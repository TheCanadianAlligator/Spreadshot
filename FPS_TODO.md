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




