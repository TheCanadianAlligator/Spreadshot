# Vibe of code
Created Sunday 12 April 2026

Talking myself through a shorter, easier to read step by step list.
If you run this through an AI I will actually fucking murder you.


* ☐ Weapon Class
	* ☑ Who owns the gun? This might be irrelevant right now, if this prototype is mostly just a gun showcase we are the only ones who get to shoot.
	* ▷ What kind of projectile are we using?
	* ☑ How fast do we get to shoot?
		* ☑ Firerate in Rounds Per Second.
		* ☑ Converts FireRate_RPS to FireRate_Timer, just because it's kind of easier to visualize for me.
			* ☐ "mr brown when are we gonna use this lol" videogames. videogames are where we're gonna use this. make sure this shit works
	* ☑ How many projectiles per shot?
		* ☐ This could be a Float for a chance to multishot, but maybe that can be a more of "exploding dice" situation somewhere else.
		* ☑ Add an "Accurate" flag + AccurateSpread value to ALL weapons for a single consistent shot and chaotic multishots.
			* ☑ AccurateShots because maybe we want a BUNCH of shots to be accurate, and the rest inaccurate.
	* ☑ How much spread does the gun have?



* ☐ Weapon types
	* ☐ Blaster for now will be very standard just to make sure things are working
		* ☐ 4 shots per second
	* ☐ Machinegun. Let's model it roughly after maybe a mix of Blood's Tommygun and Doom 64 Unmaker - that sound fun. <https://www.youtube.com/watch?v=vvkraMwjo24>
		* ☐ 600rpm = 10rps
		* ☐ Three projectiles
			* ☐ Maybe we can somehow give each projectile its own spread, so shooting centre is still consistent, but you also have shots flying at peripheral enemies.
			* ☑ Add an "Accurate" flag + AccurateSpread value to ALL weapons for a single consistent shot and chaotic multishots.
	* ☐ Shotgun - let's model this one after UT's Flak Cannon.
		* ☐ 1rps is fine.
		* ☐ At least 6 or 7 projectiles.
		* ☐ Wait do I need to add drag to the projectiles



* ☐ Projectile Class
	* ☐ Who owns the projectile?
	* ☑ What gun made the projectile?
	* ▷ How fast is our projectile?
		* ☐ Check: should this be a vector, or just a float we add to the CONSTRUCTION of a vector?
	* ☑ How quickly does it fade from existence? This is important for distinguishing short vs long range weapons.
	* ☑ How much damage does it do? (Implementing the dealing of damage can wait until we have hurtable NPCs.)
	* ☑ How much push force does it cause?
	* ☑ Does it stick to wall? Maybe we can skip this but it could make it easier to see how well things are working.
	* ☑ Does it explode?
		* ☑ How big?
	* ☐ Does it bounce off walls? This one is actually important as the Flak Cannon from Unreal works this way and it's pretty core to the "chaotic" identity to the weapons.
		* ☐ Making it bounce:
			* ☐ When the projectile hits a solid object, I guess we're inverting the vector of velocity it has right?
			* ☐ Maybe this just handles itself through physics:
				* ☐ A fast enough launch force	
				* ☐ Physical material properties
				* ☐ "Extra bouncy" flag?



* ☐ Shooting a weapon
	* ☐ On left click: 
		* ☐ Check what gun we're using
		* ☐ Make sure we're allowed to shoot right now (have we waited long enough between each shot?)
			* ☐ Tell the gun to start shooting
				* ☐ Create the gun's projectile(s) that fly towards where we're facing
					* ☐ Where is our camera pointed?
					* ☐ The bullet should come from somewhere just in front of/below the camera, and target towards either a faraway point or the closest solid intersection
					* ☐ For a split second, the bullet should not have any collision, so we don't shoot ourselves.
						* ☐ Make sure it's just short enough that we CAN blow ourselves up by shooting the floor.
			* ☐ Wait between each shot, with time variance depending on the gun we're using.
				* ☐ I want to preserve my wrists so let's just have all guns full auto (you can just hold down LMB to keep shooting). This means scheduling another shoot event as soon as the refire timer is done.
	* ☐ When we release left click, tell the gun to stop shooting.
		* ☐ Stop the timer scheduling the next shoot event.



* ☐ Changing weapon
	* ☐ on pressing 1/2/3:
		* ☐ Tell the current gun to stop shooting
		* ☐ Tell the player character to change what gun we're using
		* ☐ Tell the HUD to change what gun it says we're using




