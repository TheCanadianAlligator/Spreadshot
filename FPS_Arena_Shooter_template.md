# FPS Arena Shooter template
Created Saturday 11 April 2026

These are the notes I've taken from Unreal's default FPS Arena Shooter template. I think we're probably gonna try to replicate this in a clean FPS project rather than repurpose the Arena Shooter template as there's too many moving parts and I don't fully understand how EVERYTHING in this works.

**[+BPShooterCharacter](#FPS Arena Shooter template:BPShooterCharacter)** 
### Shoot:

* The shooter character calls the shooting, which then happens from a class called **[+ShooterWeaponBase](#FPS Arena Shooter template:ShooterWeaponBase)** (the class for the gun they are using).
* Shooting/stop shooting is called by an event called "Enhanced Input Action IA_Shoot" (Triggered/cancelled/completed) or the events "Event Touch Shoot Start" and "Event Touch Shoot End" from BPI Touch Shooter.
	* "BPI" stands for "Blueprint interface."
	* BPI Touch Shooter is in Variant_Shooter/Input. I don't know how to see inside it but judging by the name and its location I think it's for touchscreen controls.
	* For some reason we check first if this is allowed (**Get** Is Valid/Is Not Valid) and only if it's allowed, we give the **Start Firing** function (located in [+ShooterWeaponBase](#FPS Arena Shooter template:ShooterWeaponBase)) our gun and tell it to fire.
	* If it's not allowed, we give the **Stop Firing** function (also in [+ShooterWeaponBase](#FPS Arena Shooter template:ShooterWeaponBase)) our gun and tell it to stop firing.


### Swap weapon:

* [+BPShooterCharacter](#FPS Arena Shooter template:BPShooterCharacter) once again calls the swapping, either from **Enhanced Input Action IA_SwapWeapon** or a corresponding event for the touch interface.
* First we ensure we have at least one weapon.
	* Look through the character's array of *Owned Weapons***:**
	* get its length
	* If its length is greater than 1...
* Calling the weapon's **Deactivate Weapon** function (located in [+ShooterWeaponBase](#FPS Arena Shooter template:ShooterWeaponBase))...
	* Hides the weapon through **Set Actor Hidden In Game**
	* **Stop Firing**
	* Tells the owner to process deactivating the weapon:
		* **Get Owner** of itself; the owner plugs into the Target of BPI Weapon Holder's **On Weapon Deactivated**
		* Also plugs self as the weapon being deactivated in **On Weapon Deactivated**
* This FPS template only has swap forward, so we choose the next weapon in the list of **Owned Weapons**. If we've reached it, loop back to index 0.
* This next weapon is **Set** to the **Current Weapon**
* Tell the weapon to **Activate Weapon** (also in [+ShooterWeaponBase](#FPS Arena Shooter template:ShooterWeaponBase))
	* Show the weapon.
	* Tell the owner to process weapon activation.


### Damage:
The character only handles *taking* the damage.


* Event **AnyDamage** outputs Damage, Damage Type, Instigated By, and Damage Causer pins. For now we only use Damage (float).
* If we currently have HP, we subtract our current HP from the damage taken.
* Then, we update the health UI:
	* Get our current HP divided by our max HP.
	* Plug that into BPI Shooter's **Update Health UI** event's Percentage input.
	* The Player Controller is the target.
* If our health is equal or less than 0.0:
	* If our current weapon **is valid** (activated?)
		* Tell the weapon to **Deactivate Weapon**
	* Tell the BP Shooter Character to **Die.**
		* First: 
			* ~~increment the team's score.~~
				* ~~There's some stuff here about getting the gamemode and team byte~~
				* ~~and something about adding dead to some array of dead tags?~~
			* Tell the Movement Component to **Stop Movement Immediately.**
			* Tell the Movement Component to **Disable Movement.**
			* Tell the Player Controller to **Disable Input.**
			* Tell the BPI Shooter to **Update Bullet UI.**
		* Second:
			* Tell the Mesh to **Set Collision Profile** to Ragdoll, and Update Overlaps (true).
			* Tell the Mesh to **Simulate Physics** (true).
			* Tell the Actor to **Deactivate** the First Person Camera.
			* Tell the Actor to **Activate** the Death Camera.
		* Third:
			* After a **Delay** of 5.0 seconds,
			* **Destroy Actor** (self).
		* (Sidenote: a sequence seems to be a neat way to do this without spaghetti-ing a line down all the way down and to the left.)


So what does the damage? In this template, the [+ShooterProjectileBullet](#FPS Arena Shooter template:ShooterProjectileBullet) - which is fired by a [+ShooterWeaponBase](#FPS Arena Shooter template:ShooterWeaponBase) 

We begin to shoot the gun with the **Start Firing** custom event.

* First, **Set** the Is Firing boolean to TRUE, regardless of whether we actually shoot.
* Check if enough time has passed since the last time we shot:
	* **Get Game Time in Seconds**, and **Subtract** it by the Time of Last Shot.
	* If this time is **Greater Than** the Refire Rate of the weapon...
		* We can **Fire** right away (call the event!)
	* Otherwise, schedule the next shot...
		* **Create Event** to **Fire**.
		* Plug that into a **Set Timer by Event** using our refire rate as the time for a timer**.**
		* **Set** the Refire Timer.



* Custom Event: **Fire**
* Check if we are still firing: is the Is Firing boolean true?
* Find out where the weapon holder is, through returning **Get Owner** of self, and plugging it into **Get Weapon Target Location** of BPI Weapon Holder.
	* Interesting! This seems to be a function within BP_ShooterCharacter.
	* We aim forward from the First Person Camera of the actor:
		* **Get World Location** of the First Person Camera.
		* **Get Forward Vector** of the First Person Camera, and then multiply by 10000.0. 
			* Oh, this is like a Very Far Away point that marks the endpoint of our line.
	* Check for obstacles occluding line of sight with **Line Trace By Channel:**
		* Plug the world location into Start.
		* Add the world location and the multiplied forward vector, and plug that into End.
	* The *Out Hit* result plugs into a **Break Hit Result.** 
		* This plugs into a **Select Vector** function:
			* Option *A* is the *Impact Point;* This is the first solid object that this line trace hits.
			* Option *B* is the *Trace End;* This is the 10000.0 endpoint of the big line we drew.
			* Plug the *Blocking Hit* boolean into the *Pick A* boolean: if the line hit something that blocks it, we choose Option A (the impact point) instead of Option B (the point far away).
			* This should make projectile-based attacks make a lot more sense when fired from a non-centralized location (e.g. a typical right hand "hipfire" location).
		* **Return** whatever this Select Vector outputs.
* **Fire Bullet** from [+ShooterWeaponBase](#FPS Arena Shooter template:ShooterWeaponBase) using this target location.
	* First, **Spawn Actor.** This is the bullet being fired.
		* Plug in: 
			* the *Bullet Class* (what kind of bullet we are making).
			* **Get Owner** for the Owner. I'm pretty sure this is the owner of the gun??
			* The *Pawn Owner* is the Instigator. I *think* this the person who *shot* the gun? I'm not sure why we call this twice, but maybe instigator is specifically important for later.
			* Where the bullet is coming from (our target location -> a function called **Bullet Spawn Transform**)
				* First, we take the mesh of the first person version of the weapon, and find its muzzle, using **Get Socket Location.**
				* We use **Find Look at Rotation** to figure out which way the player is looking (i think?) by comparing the gun's muzzle location to where the player is looking; the *Target Position* from the **Line Trace By Channel** earlier.
					* Plug the resulting rotation value into **Get Forward Vector** to get a vector we can work with; multiply this by 10, and add the muzzle's location to it.
						* This plugs into a **Make Transform**, as the *Location* where the bullet should spawn from.
					* At the same time, run a **Random Unit Vector** and multiply the result by the gun's *Aim Variance.* 
						* Add it to the camera's *Target Position* and plug this in as the *Target* of a **Find Look at Rotation**. 
					* Use the same value you found for the **Make Transform**'s origin as the start location for this **Find Look at Rotation;** plug the return value into the **Make Transform**'s *Rotation* - which way the bullet will deviate from your aim.
				* The function **Return**s a Transform.
	* Second, animate the gun firing. We'll skip this for now, as it doesn't have to be pretty.
		* Play the montage that contains our FirstPersonFire Animation.
		* Add slight pitch to simulate recoil.
	* Third, the player loses bullets. 
		* First, we actually lose the bullet.
			* Decrement the *Current Bullets* count by 1 (-- function)
			* If the mag is empty, reload;
				* If *Current Bullets-- <=* 0:
					* Set *Current Bullets* to *Mag Size.*
		* Second, we update the weapon HUD to reflect that.
			* Have the BPI Weapon Holder Update Weapon HUD, using:
				* **Get Owner** for the Owner, whoever's holding the gun.
				* *Current Bullets*
				* *Mag Size*
		* Save the time of the next shot so you can't just spam LMB to shoot faster than you should:
	* **Get Game Time in Seconds** for the current time.
	* **Set** Time of Last Shot.
* There's a perception system noise so enemies can hear your shot. Look into this later if you're interested.
	* It has something to do with the gun's loudness, who owns the gun, and who+where the shot came from.
* Check if we're full auto...
	* If true: schedule the next shot.
		* **Create Event** to **Fire**.
		* Plug that into a **Set Timer by Event** using our refire rate as the time for a timer**.**
		* **Set** the Refire Timer.
	* If false: schedule the cooldown.
		* **Create Event** that tells us **Fire Cooldown Expired.**
		* Plug that into a **Set Timer by Event** using our refire rate as the time for the timer.
		* **Set** the Refire Timer.


Semi-auto guns can fire again after the **Fire Cooldown Expired** event activates.

* **Get Owner** (whoever holds the gun) and plug it into BPI Weapon Holder's **On Semi Weapon Refire** event.



We **Stop Firing** by giving the gun a custom event to, um, **Stop Firing.**

* **Set** the *Is Firing* boolean to FALSE.
* **Clear and Invalidate Timer by Handle:** targetting *Refire Timer.*
	* (this is the timer we set in the **Start Firing** and **Fire** events to shoot a full auto gun's next bullet.)



# BPShooterCharacter

# ShooterProjectileBullet

# ShooterWeaponBase
Created Saturday 11 April 2026

This seems to be a class for weapons. 
Weapons use this class as a base and tweak variables to become different.

There's a pistol, assault rifle, and grenade launcher. 
> All weapons featured have different fire rates and ammo capacities.
> The pistol and assault rifle fire darts, but the grenade launcher fires exploding orbs.
> > The projectiles themselves (ShooterProjectileBullet) store their own stats for things like damage, velocity, impact force.
> > > They also have Explosion Radius and a flag to Explode on hit, the flag is FALSE by default.

When a gun is created (event **BeginPlay):**

* Prepare for cleanup later by listening to On Destroyed events on the owner.
	* **Get Owner** (whoever's holding the gun?) and create the Event **On Weapon Owner Destroyed(DestroyedActor)** to be triggered by **Bind Event to On Destroyed.**
	* Attach the weapon's meshes to the owner:
		* **Attach Weapon Meshes** in BPI Weapon Holder.
			* It asks for a Target (weapon's owner), Weapon (self), First Person Mesh, and Third Person Mesh.
	* Have the weapon's owner be **Cast To Pawn,** and **Set** it as the *Pawn Owner* variable as a reference for later.


We begin to shoot the gun with the **Start Firing** custom event.

* First, **Set** the Is Firing boolean to TRUE, regardless of whether we actually shoot.
* Check if enough time has passed since the last time we shot:
	* **Get Game Time in Seconds**, and **Subtract** it by the Time of Last Shot.
	* If this time is **Greater Than** the Refire Rate of the weapon...
		* We can **Fire** right away (call the event!)
	* Otherwise, schedule the next shot...
		* **Create Event** to **Fire**.
		* Plug that into a **Set Timer by Event** using our refire rate as the time for a timer**.**
		* **Set** the Refire Timer.


**Fire** a bullet:

* Check if we are still firing: is the Is Firing boolean true?
* Find out where the weapon holder is, through returning **Get Owner** of self, and plugging it into **Get Weapon Target Location** of BPI Weapon Holder.
* **Fire Bullet** from [ShooterWeaponBase](#FPS Arena Shooter template:ShooterWeaponBase) using this target location.
* Save the time of the next shot so you can't just spam LMB to shoot faster than you should:
	* **Get Game Time in Seconds** for the current time.
	* **Set** Time of Last Shot.
* There's a perception system noise so enemies can hear your shot. Look into this later if you're interested.
	* It has something to do with the gun's loudness, who owns the gun, and who+where the shot came from.
* Check if we're full auto...
	* If true: schedule the next shot.
		* **Create Event** to **Fire** (again).
		* Plug that into a **Set Timer by Event** using our refire rate as the time for a timer**.**
		* **Set** the Refire Timer.
	* If false: schedule the cooldown.
		* **Create Event** that tells us **Fire Cooldown Expired.**
		* Plug that into a **Set Timer by Event** using our refire rate as the time for the timer.
		* **Set** the Refire Timer.


Semi-auto guns can fire again after the **Fire Cooldown Expired** event activates.

* **Get Owner** (this is whoever's holding the gun) and plug it into BPI Weapon Holder's **On Semi Weapon Refire** event.


We **Stop Firing** by giving the gun a custom event to, um, **Stop Firing.**

* **Set** the *Is Firing* boolean to FALSE.
* **Clear and Invalidate Timer by Handle:** targetting *Refire Timer.*
	* (this is the timer we set in the **Start Firing** and **Fire** events to shoot a full auto gun's next bullet.)




* **Fire Bullet**...
	* First, **Spawn Actor.** This is the bullet being fired.
		* Plug in: 
			* the *Bullet Class* (what kind of bullet we are making).
			* Where the bullet is coming from (our target location -> Spawn Transform)
			* **Get Owner** for the Owner. I'm pretty sure this is the owner of the gun??
			* The *Pawn Owner* is the Instigator. I *think* this the person who *shot* the gun? I'm not sure why we call this twice, but maybe instigator is specifically important for later.
	* Second, animate the gun firing. We'll skip this for now, as it doesn't have to be pretty.
		* Play the montage that contains our :FirstPersonFire Animation.
		* Add slight pitch to simulate recoil.
	* Third, the player loses bullets. 
		* First, we actually lose the bullet.
			* Decrement the *Current Bullets* count by 1 (-- function)
			* If the mag is empty, reload;
				* If *Current Bullets <=* 0:
					* Set *Current Bullets* to *Mag Size.*
		* Second, we update the weapon HUD to reflect that.
			* Have the BPI Weapon Holder Update Weapon HUD, using:
				* **Get Owner** for the Owner - whoever's holding the gun.
				* *Current Bullets*
				* *Mag Size*




* **Deactivate Weapon** 
	* Hides the weapon through **Set Actor Hidden In Game**
	* **Stop Firing**
	* Tells the owner to process deactivating the weapon:
		* **Get Owner** of itself; the owner plugs into the Target of BPI Weapon Holder's **On Weapon Deactivated**
		* Also plugs self as the weapon being deactivated in **On Weapon Deactivated**



* **Activate Weapon** 
	* Show the weapon.
	* Tell the owner to process weapon activation.


 **Calculate** **Bullet Spawn Transform**

* First, we take the mesh of the first person version of the weapon, and find its muzzle, using **Get Socket Location.**
* We use **Find Look at Rotation** to figure out which way the player is looking (i think?) by comparing the gun's muzzle location to where the player is looking; the *Target Position* from the **Line Trace By Channel** earlier.
	* Plug the resulting rotation value into **Get Forward Vector** to get a vector we can work with; multiply this by 10, and add the muzzle's location to it.
		* This plugs into a **Make Transform**, as the *Location* where the bullet should spawn from.
	* At the same time, run a **Random Unit Vector** and multiply the result by the gun's *Aim Variance.* 
		* Add it to the camera's *Target Position* and plug this in as the *Target* of a **Find Look at Rotation**. 
	* Use the same value you found for the **Make Transform**'s origin as the start location for this **Find Look at Rotation;** plug the return value into the **Make Transform**'s *Rotation* - which way the bullet will deviate from your aim.
* The function **Return**s a Transform.

# Vibe of code
Created Sunday 12 April 2026

Talking myself through a shorter, easier to read step by step list.
If you run this through an AI I will actually fucking murder you.


* ☐ Weapon
	* ☐ Who owns the gun? This might be irrelevant right now, if this prototype is mostly just a gun showcase we are the only ones who get to shoot.
	* ☐ What kind of projectile are we using?
	* ☐ How fast do we get to shoot?
	* ☐ How many projectiles per shot?
	* ☐ How much spread does the gun have?



* ☐ Projectile
	* ☐ Who owns the projectile?
	* ☐ What gun made the projectile?
	* ☐ How fast is our projectile?
	* ☐ How quickly does it fade from existence? This is important for distinguishing short vs long range weapons.
	* ☐ How much damage does it do? (Implementing the dealing of damage can wait until we have hurtable NPCs.)
	* ☐ How much push force does it cause?
	* ☐ Does it stick to wall? Maybe we can skip this but it could make it easier to see how well things are working.
	* ☐ Does it explode?
		* ☐ How big?
	* ☐ Does it bounce off walls? This one is actually important as the Flak Cannon from Unreal works this way and it's pretty core to the "chaotic" identity to the weapons.
		* ☐ Making it bounce:
			* ☐ When the projectile hits a solid object, I guess we're inverting the vector of velocity it has right?
			* ☐ Maybe this just handles itself through physics:
				* ☐ A fast enough launch force	
				* ☐ Physical material properties



* ☐ Shooting a weapon
	* ☐ On left click: 
		* ☐ Check what gun we're using
		* ☐ Make sure we're allowed to shoot right now (have we waited long enough between each shot?)
			* ☐ Tell the gun to start shooting
				* ☐ Create the gun's projectile(s) that fly towards where we're facing
					* ☐ Where is our camera pointed?
					* ☐ The bullet should come from somewhere just in front of/below the camera, and target towards either a faraway point or the closest solid intersection
			* ☐ Wait between each shot, with time variance depending on the gun we're using.
				* ☐ I want to preserve my wrists so let's just have all guns full auto (you can just hold down LMB to keep shooting). This means scheduling another shoot event as soon as the refire timer is done.
	* ☐ When we release left click, tell the gun to stop shooting.
		* ☐ Stop the timer scheduling the next shoot event.



* ☐ Changing weapon
	* ☐ on pressing 1/2/3:
		* ☐ Tell the current gun to stop shooting
		* ☐ Tell the player character to change what gun we're using
		* ☐ Tell the HUD to change what gun it says we're using




