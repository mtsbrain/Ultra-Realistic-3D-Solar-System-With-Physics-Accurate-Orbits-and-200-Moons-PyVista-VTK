A fully dynamic, physics-driven 3D Solar System simulation built using PyVista, VTK, NumPy, and real astronomical parameters.
This project features:
* Accurate orbital mechanics (Keplerian elements from JPL Horizons)
* True axial tilts & rotation periods for planets and moons
* 150+ moons with individual orbital elements
* Dynamic zoom limits per object
* Realistic irregular potato-shaped bodies
* Saturn’s ring system
* Comets with real orbital paths
* Scaled radii & distances for stable visualization
* High-performance rendering with level-of-detail logic
<img width="1358" height="663" alt="image" src="https://github.com/user-attachments/assets/5dbf978b-78ad-4503-ace0-a582004380af" />
Full Controls Overview
Navigate the simulation interactively with these keyboard shortcuts (case-sensitive; press once to activate). Focus on a planet to unlock moon viewing (numeric keys). Mouse: Left-drag to rotate view, right-drag to pan, wheel to zoom (clamped for clipping prevention). Arrow keys: Pan/orbit camera (use for gradual full-system zoom-out to reveal tiny outer moons without clamping issues—wheel zoom may obscure distant objects).
Key,Action,Details:

* t-Focus Sun,Centers camera on Sun (min zoom: ~5 units)
* y-Focus Mercury,Zooms to innermost planet (no moons)
* v-Focus Venus,Targets cloudy world (no moons)
* g-Focus Earth,Homes in on home planet; press 1 for Luna
* m-Focus Mars,"Views Red Planet; press 1 for Phobos, 2 for Deimos"
* j-Focus Jupiter,Gas giant view; press 1–30+ for 95 moons (inner: Metis=1 to Callisto=8; outer: tiny irregulars)
* s-Focus Saturn,"Ringed wonder; press 1–83 for moons (e.g., 1=Mimas, 20=Titan)"
* u-Focus Uranus,"Sideways spinner; press 1–27 for moons (e.g., 1=Cordelia, 15=Titania)"
* n-Focus Neptune,"Windy blue; press 1–14 for moons (e.g., 1=Naiad, 8=Triton)"
* o-Focus Pluto,Dwarf at edge; no moons here (Charon via separate logic)
* i-Focus Eris,Distant dwarf (if enabled)
* h-Focus Haumea,Elongated Kuiper object
* k-Focus Makemake,Icy world
* c-Focus Ceres,Asteroid belt dwarf
* a-Focus Asteroid Belt,Pans to main belt (~2.7 AU); reveals thousands of points
* z-Focus Halley Comet,Tracks famous periodic comet
* x-Focus Hale-Bopp,Great Comet of 1997
* l-Focus Encke,Short-period comet
* d-Focus Lovejoy,Recent sungrazer
* 0–9(Numerics),Select Moon,"After planet focus (e.g., j then 1,2,3 for Jupiter's moon #1,2,3); buffers multi-digit"
* Arrow Keys,Pan/Orbit Camera,Smooth full-system navigation; ideal for unveiling tiny moons (avoids wheel clamp)
* ESC,Pause/Exit,Halts animation or closes viewer

* Pro Tip: For crisp views of faint outer moons (e.g., Jupiter's 95+), start at Sun focus (t), then arrow-key zoom out to ~30 AU—wheel clamping keeps close-ups sharp but hides peripherals

SUN
<img width="1364" height="665" alt="image" src="https://github.com/user-attachments/assets/1a4be5fb-58ce-4a51-8cf3-6805d13dae0d" />
The Sun serves as the central gravitational anchor and visual centerpiece of this solar system simulation, rendered as a glowing, rotating sphere using PyVista. Its implementation draws from real astronomical data (e.g., NASA/JPL sources) for authenticity, while incorporating dynamic effects like barycentric wobble due to planetary influences. Below is a breakdown of its key features, parameters, and behaviors.
* Radius: 696,340 km (scaled to ~0.02x relative to orbital distances for visualization; SUN_RADIUS = REAL_SUN_RADIUS_KM * SIZEKM_TO_SCENE).
* Gravitational Parameter (GM): 1.327 × 10¹¹ km³/s² (GM_SUN), used for orbital calculations of all bodies.
* Rotation Period: ~25 Earth days at the equator (differential rotation approximated; ROTATION_PERIOD["sun"] = 25.0 * 86400 seconds), converted to ~0.000166° per simulated second (ROTATION_DEG_PER_SIMSEC["sun"]).
* Axial Tilt: 7.25° relative to the ecliptic plane (AXIAL_TILTS["sun"]), applied via actor.RotateX(AXIAL_TILTS["sun"]) for realistic obliquity.
These values ensure Keplerian orbits and rotations align with J2000 epoch standards, with positions computed dynamically via kepler_to_state functions.

* Rendering: Created as a pv.Sphere with a bright yellow/orange color (color="yellow", opacity=0.8) and optional PBR shading for a photospheric glow (pbr=True, roughness=0.1). Textures (e.g., from examples.load_globe_texture()) can be layered for surface details like sunspots.
* Barycentric Wobble: The Sun isn't static—it "wobbles" slightly due to Jupiter and Saturn's pull, calculated in the update() loop.
  
* Rotation: Continuously rotates around its Z-axis in the animation loop (actor.RotateZ(deg % 360.0)), simulating solar day-night cycles (though simplified, as the Sun lacks a solid surface).
* Role in Simulation: Acts as the reference point for all distances (e.g., comet sublimation at 3 AU via SUBLIMATION_DISTANCE), with its position converted to km-scale for heliocentric calcs (sun_pos_km = sun_pos / KM_TO_SCENE).
* Focus on Sun: Press t to center the camera on the Sun (focus_sun() method), with automatic clipping range reset for smooth zooming (min distance: ~5 units).

This implementation balances performance (e.g., no heavy volumetric effects on the Sun) with educational value, making it easy to visualize solar dominance in the system. For enhancements, consider adding solar flares via particle systems or real-time data integration from NASA's Solar Dynamics Observatory (SDO).



MERCURY
<img width="1365" height="659" alt="image" src="https://github.com/user-attachments/assets/b926ef78-9f44-4bff-a8f6-a5d48f29ce81" />
Mercury, the smallest and innermost planet in our solar system, orbits closest to the Sun at an average distance of 0.387 AU. As a rocky terrestrial world, it boasts a heavily cratered, Moon-like surface scarred by ancient impacts, with vast plains of cooled lava and steep escarpments. Notably, Mercury lacks a true atmosphere; instead, it maintains a razor-thin exosphere—a diffuse envelope of scattered atoms primarily consisting of oxygen (O₂, ~42%), sodium (Na, ~29%), hydrogen (H₂, ~22%), helium (He, ~6%), and potassium (K)—continuously replenished by solar wind bombardment and micrometeorite strikes. This exosphere provides negligible insulation, exposing the planet to brutal temperature swings from a scorching 427°C (800°F) during the day to a frigid -173°C (-280°F) at night, and offers scant protection from solar radiation. NASA's Messenger mission (2008–2015) revealed these details, highlighting Mercury's iron-rich core (making up ~75% of its mass) and 3:2 spin-orbit resonance, where it rotates three times for every two orbits.
In this simulation, Mercury is modeled with high fidelity using Keplerian orbital elements and real-time dynamics, emphasizing its slow rotation and proximity to the Sun for educational visualization.
<img width="1365" height="680" alt="image" src="https://github.com/user-attachments/assets/c7dd0d4c-5178-4a62-91aa-606eb0f6fb2f" />

* Radius: 2,440 km (scaled to MERCURY_RADIUS = REAL_MERCURY_RADIUS_KM * SIZEKM_TO_SCENE for scene fitting).
* Gravitational Parameter (GM): 2.2032 × 10⁴ km³/s² (GM_MERCURY), influencing potential moon simulations (though Mercury has none).
* Orbital Distance: 0.387 AU (MERCURY_AU), yielding MERCURY_ORBIT_RADIUS = MERCURY_AU * AU_SCALE.
* Rotation Period: 58.646 Earth days (sidereal; ROTATION_PERIOD["mercury"] = 58.646 * 86400 seconds), resulting in ~0.000066° per simulated second (ROTATION_DEG_PER_SIMSEC["mercury"])—reflecting its 3:2 resonance (not fully simulated here for simplicity).
* Axial Tilt: 0.03° (AXIAL_TILTS["mercury"]), nearly equatorial, applied via actor.RotateX(AXIAL_TILTS["mercury"]).
* Rendering: Depicted as a grayish sphere (pv.Sphere with color="gray", opacity=1.0) textured to mimic its regolith-covered, cratered terrain (e.g., via procedural noise or Messenger imagery). No atmospheric effects are added due to its exosphere's sparsity, but subtle glow halos can be toggled for solar proximity visualization.
  <img width="1364" height="660" alt="image" src="https://github.com/user-attachments/assets/32576a2e-842c-42b2-bdce-ad1f4e486b1b" />

* Orbital Motion: Updated dynamically in the update() loop using osculating elements: set_body_position_from_elements("mercury", self.mercury_actor, orbital_elements["mercury"], target_jd). This computes heliocentric position in km, scaled to scene units (km_to_scene), with smoothing for fluid animation.
* Rotation: Slowly spins prograde around its Z-axis (actor.RotateZ(deg % 360.0)), with initial orientation adjusted for Greenwich Mean Sidereal Time (GMST) alignment:merc_gmst = gmst_deg * (ROTATION_PERIOD["earth"] / abs(ROTATION_PERIOD["mercury"])) % 360.0
self.mercury_actor.RotateZ(-merc_gmst).
* Focus on Mercury: Press y to center the camera (set_planet_focus('mercury', self.mercury_actor)), auto-adjusting clipping for close-up crater views (min distance: ~MERCURY_RADIUS * 20 units).

This module captures Mercury's harsh, airless isolation, ideal for demonstrating extreme orbital mechanics. Future updates could integrate real-time solar wind data from NASA's ACE satellite for dynamic exosphere effects.

VENUS
<img width="1364" height="671" alt="image" src="https://github.com/user-attachments/assets/cb6360c0-ec50-423f-ac80-74dd55660d15" />
Venus, often dubbed Earth's "evil twin" due to its similar size and rocky composition, is the second planet from the Sun at an average distance of 0.723 AU. Blanketed by a dense, toxic atmosphere composed mainly of carbon dioxide (96.5%) with traces of nitrogen (3.5%) and sulfuric acid clouds, it experiences a runaway greenhouse effect that traps heat, making it the hottest planet in the solar system with surface temperatures averaging 464°C (867°F)—hot enough to melt lead. Atmospheric pressure at the surface is a crushing 92 times that of Earth, rendering the landscape a hellish vista of volcanic plains, massive lava domes, and continent-sized highlands like Ishtar Terra. Its rotation is retrograde and excruciatingly slow (243 Earth days per spin, longer than its 225-day orbit), resulting in a "day" longer than its year, while radar mapping from NASA's Magellan mission (1989–1994) unveiled a surface dominated by volcanism and few impact craters. Venus's thick cloud cover reflects 70% of sunlight, giving it a brilliant white appearance from Earth, but hides a dynamic world probed by ongoing missions like Japan's Akatsuki (2015–present).
In this simulation, Venus is rendered with emphasis on its retrograde spin and atmospheric opacity, using Keplerian dynamics for precise orbital tracking and simplified cloud visuals for educational impact.
<img width="1365" height="670" alt="image" src="https://github.com/user-attachments/assets/f72758b6-b9e5-495d-a0e6-843c116c978f" />
* Radius: 6,052 km (scaled to VENUS_RADIUS = REAL_VENUS_RADIUS_KM * SIZEKM_TO_SCENE for relative sizing).
* Gravitational Parameter (GM): 3.249 × 10⁵ km³/s² (GM_VENUS), for potential future enhancements like surface simulations.
* Orbital Distance: 0.723 AU (VENUS_AU), yielding VENUS_ORBIT_RADIUS = VENUS_AU * AU_SCALE.
* Rotation Period: -243.025 Earth days (retrograde; ROTATION_PERIOD["venus"] = -243.025 * 86400 seconds), yielding ~ -0.000059° per simulated second (ROTATION_DEG_PER_SIMSEC["venus"])—the negative sign enforces backward spin.
* Axial Tilt: 177.4° (AXIAL_TILTS["venus"]), effectively a 2.6° tilt in the opposite direction due to retrograde motion, applied via actor.RotateX(AXIAL_TILTS["venus"]).
* Rendering: Modeled as a creamy-yellow sphere (pv.Sphere with color="yellow" or "orange" for cloud reflection, opacity=0.9) with optional procedural textures simulating sulfuric acid haze (e.g., layered noise for cloud bands). A faint atmospheric halo (add_atmosphere_halo) can represent the upper cloud deck, with opacity scaled to mimic 99% light scattering.
* Orbital Motion: Dynamically positioned in the update() loop via osculating elements: set_body_position_from_elements("venus", self.venus_actor, orbital_elements["venus"], target_jd). Converts heliocentric km coordinates to scene units (km_to_scene), with velocity smoothing for seamless animation.
<img width="1365" height="663" alt="image" src="https://github.com/user-attachments/assets/e0243889-53f5-4ce4-8626-079cfb084e44" />

* Rotation: Retrograde spin around the Z-axis (actor.RotateZ(-deg % 360.0) to account for negative period), initialized with GMST-adjusted orientation: venus_gmst = gmst_deg * (ROTATION_PERIOD["earth"] / abs(ROTATION_PERIOD["venus"])) % 360.0
self.venus_actor.RotateZ(-venus_gmst).
* Atmospheric Effects: While not volumetrically simulated (for performance), proximity to the Sun triggers subtle glow enhancements in halo logic (r_km = np.linalg.norm(pos_km - sun_pos_km)), evoking Venus's phase changes and superior conjunction visibility.
* Venus's implementation highlights its paradoxical "slow dance" with the Sun, keeping compute overhead low without moons.
* Focus on Venus: Press v to zoom in (set_planet_focus('venus', self.venus_actor)), with clipping auto-reset for detailed cloud layer views (min distance: ~VENUS_RADIUS * 20 units).

This captures Venus's veiled, volcanic fury, perfect for illustrating greenhouse extremes. Enhancements could include dynamic cloud rotation from Akatsuki data.

EARTH
<img width="1358" height="648" alt="image" src="https://github.com/user-attachments/assets/0c10b204-4eeb-4b8b-965c-41cad178cbc9" />
Earth, our blue marble and the only known cradle of life in the solar system, orbits at a comfortable 1 AU from the Sun, enabling a temperate climate that supports diverse ecosystems across seven continents, vast oceans covering 71% of its surface, and a protective atmosphere of nitrogen (78%) and oxygen (21%) that shields against cosmic rays while driving dynamic weather patterns like hurricanes and auroras. This gaseous envelope, laced with water vapor and trace gases, fosters the greenhouse effect that keeps global averages at a life-sustaining 15°C (59°F), though human-induced changes are accelerating warming. Earth's iron-nickel core generates a magnetic field deflecting solar wind, while its active geology—plate tectonics, volcanoes, and earthquakes—recycles its crust. Iconic missions like Apollo (1969–1972) and ongoing observations from satellites such as Terra and Aqua have mapped its ever-changing visage, revealing seasonal ice caps, swirling cloud vortices, and bioluminescent oceans under the night side.
In this simulation, Earth is the interactive hub, complete with axial precession, lunar orbit, and atmospheric glow, leveraging Keplerian elements for synchronized day-night cycles and tidal locking visuals.
<img width="1358" height="652" alt="image" src="https://github.com/user-attachments/assets/79858c6e-650d-4f77-b4e1-07a6432c35f7" />
A standout feature is Luna (Latin for Moon), Earth's sole natural satellite and the fifth-largest in the solar system, orbiting at an average 384,400 km with a diameter of 3,475 km—about a quarter of Earth's. Formed ~4.5 billion years ago from debris of a Mars-sized impactor (Theia), Luna stabilizes Earth's axial tilt for consistent seasons, drives tides influencing marine life and coastal erosion, and once hosted a molten magma ocean now solidified into basaltic maria (dark "seas") and rugged highlands pocked by craters like Tycho. Its thin exosphere (mostly argon and helium) and lack of atmosphere expose it to micrometeorites, creating perpetual "moonquakes" from tidal stresses. NASA's Artemis program (2020s) aims to return humans, building on Apollo's legacy of 382 kg of lunar samples.
<img width="1365" height="666" alt="image" src="https://github.com/user-attachments/assets/6db37606-291d-47f1-8c4b-81e51f6c1cac" />

* Radius: 6,371 km (scaled to EARTH_RADIUS = REAL_EARTH_RADIUS_KM * SIZEKM_TO_SCENE for balanced proportions).
* Gravitational Parameter (GM): 3.986 × 10⁵ km³/s² (GM_EARTH), central to moon orbital mechanics.
* Orbital Distance: 1.0 AU (EARTH_AU), defining EARTH_ORBIT_RADIUS = EARTH_AU * AU_SCALE as the baseline unit.
* Rotation Period: 0.99726968 Earth days (sidereal; ROTATION_PERIOD["earth"] = 0.99726968 * 86400 seconds), equating to ~0.004° per simulated second (ROTATION_DEG_PER_SIMSEC["earth"]) for diurnal motion.
* Axial Tilt: 23.44° (AXIAL_TILTS["earth"]), driving seasons via actor.RotateX(AXIAL_TILTS["earth"]).
* Rendering: A vibrant blue-green pv.Sphere (color="blue" with landmasses in green/brown via texture mapping, e.g., NASA's Blue Marble composite), overlaid with a translucent atmospheric halo (add_atmosphere_halo) simulating ozone layer scattering (opacity ~0.2 for Rayleigh blue skies). Clouds could be procedurally added as semi-transparent meshes for weather dynamism.
  <img width="1352" height="656" alt="image" src="https://github.com/user-attachments/assets/b175ed37-4b97-47b5-96fc-86d496c182d8" />

* Orbital Motion: Real-time heliocentric positioning in update(): set_body_position_from_elements("earth", self.earth_actor, orbital_elements["earth"], target_jd). Scaled from km to scene coordinates (km_to_scene), with inertial frame smoothing to mimic elliptical path.
* Rotation: Prograde spin synchronized to sidereal day (actor.RotateZ(deg % 360.0)), initialized with GMST for accurate longitude.
* Focus on Earth: Press g to home in (set_planet_focus('earth', self.earth_actor)), enabling moon navigation via numeric keys (1 for Luna; min distance: ~EARTH_RADIUS * 20 units).

MARS
<img width="1365" height="655" alt="image" src="https://github.com/user-attachments/assets/52da038b-17bc-46a7-9a22-c22756a929a1" />
Mars, the enigmatic Red Planet, orbits at 1.524 AU from the Sun, its rusty hue stemming from iron oxide (rust) dust blanketing vast deserts, towering canyons like Valles Marineris (four times Grand Canyon's length), and the solar system's largest volcano, Olympus Mons (22 km high). A thin atmosphere—mostly carbon dioxide (95.3%) with nitrogen (2.7%) and argon (1.6%)—creates wispy clouds, global dust storms that can engulf the planet for months, and seasonal polar ice caps of water and CO₂ frost, hinting at ancient rivers and potential subsurface oceans. Surface temperatures swing from -60°C ( -76°F) average to 20°C (68°F) highs, with low pressure (0.6% of Earth's) making liquid water unstable. Pioneering missions like Viking (1976), Pathfinder (1997), and NASA's Perseverance rover (2021–present) have uncovered organic molecules and methane plumes, fueling dreams of past microbial life and future human exploration.
In this simulation, Mars is vividly animated with its two potato-shaped moons, dust-red terrain, and orbital eccentricity, using Keplerian mechanics to showcase its elliptical path and synodic oppositions.
<img width="1351" height="659" alt="image" src="https://github.com/user-attachments/assets/783afb5c-0600-4da8-aae3-02345e188ed3" />
* Radius: 3,390 km (scaled to MARS_RADIUS = REAL_MARS_RADIUS_KM * SIZEKM_TO_SCENE for compact visualization).
* Gravitational Parameter (GM): 4.282 × 10⁴ km³/s² (GM_MARS), governing Phobos and Deimos orbits.
* Orbital Distance: 1.524 AU (MARS_AU), producing MARS_ORBIT_RADIUS = MARS_AU * AU_SCALE.
* Rotation Period: 1.025957 Earth days (sidereal; ROTATION_PERIOD["mars"] = 1.025957 * 86400 seconds), translating to ~0.0039° per simulated second (ROTATION_DEG_PER_SIMSEC["mars"]) for sol-like days.
* Axial Tilt: 25.19° (AXIAL_TILTS["mars"]), similar to Earth's for comparable seasons, via actor.RotateX(AXIAL_TILTS["mars"]).
* Rendering: A reddish pv.Sphere (color="red" or "maroon" with procedural craters and polar caps via noise textures, opacity=1.0), accented by a sparse atmospheric halo (add_atmosphere_halo) for dust storm hints—opacity modulated by solar distance (r_km = np.linalg.norm(pos_km - sun_pos_km)).
* Orbital Motion: Updated in the update() loop with osculating elements: set_body_position_from_elements("mars", self.mars_actor, orbital_elements["mars"], target_jd).  Scaled heliocentric positions (km_to_scene) with eccentricity-driven speed variations for realistic perihelion rushes.
* Rotation: Prograde axial spin (actor.RotateZ(deg % 360.0)), GMST-initialized for areographic accuracy.
  <img width="1353" height="657" alt="image" src="https://github.com/user-attachments/assets/b114dc97-0adc-4dea-819d-dc959788122d" />

* Moons and Atmosphere: Phobos (press 1) and Deimos (press 2) orbit in moon_actors["mars"], tidally locked with faint exosphere glows.
JUPITER
Jupiter, the colossal gas giant and king of the solar planets, reigns at 5.2 AU from the Sun, its swirling bands of hydrogen (90%) and helium (10%) clouds—stained ochre, white, and brown by ammonia crystals and water vapor—harboring ferocious storms like the Great Red Spot, a hurricane twice Earth's diameter raging for centuries. Composed mostly of primordial solar nebula gas, it boasts a massive rocky/icy core shrouded in metallic hydrogen layers generating ferocious magnetic fields and auroras brighter than the Sun's. With no solid surface, temperatures plummet from 165 K (-108°C) in the clouds to scorching 24,000 K at the core, while its 79 known moons (95 modeled here) range from volcanic Io to icy Europa (a prime astrobiology target with subsurface ocean) and cratered Callisto. NASA's Pioneer (1973), Galileo (1995–2003), and Juno (2016–present) missions unveiled its turbulent atmosphere, faint dusty rings from moon ejecta, and role as a "cosmic vacuum" deflecting comets—potentially preserving inner planets like Earth.
<img width="1352" height="657" alt="image" src="https://github.com/user-attachments/assets/6c6bea53-3e73-49e1-a6d0-c6d541e8f22b" />
In this simulation, Jupiter dominates with its 95 dynamically orbiting moons (LOD-culled for performance), banded sphere, and subtle ring system, employing Keplerian elements to illustrate its rapid orbit and the resonant dances of its Galilean satellites.
* Radius: 71,492 km (scaled to JUPITER_RADIUS = REAL_JUPITER_RADIUS_KM * SIZEKM_TO_SCENE for imposing presence)
* Gravitational Parameter (GM): 1.266 × 10⁸ km³/s² (GM_JUPITER), dominating moon and ring dynamics
* Orbital Distance: 5.2 AU (JUPITER_AU), yielding JUPITER_ORBIT_RADIUS = JUPITER_AU * AU_SCALE
* Rotation Period: 9.925 hours (sidereal; ROTATION_PERIOD["jupiter"] = 9.925 * 3600 seconds), spinning ~0.036° per simulated second (ROTATION_DEG_PER_SIMSEC["jupiter"])—fastest in the system, flattening its equator
<img width="1351" height="654" alt="image" src="https://github.com/user-attachments/assets/d05913ad-f2ef-4617-95ba-55ba40a4588f" />

* Axial Tilt: 3.13° (AXIAL_TILTS["jupiter"]), minimal for stable belts, applied via actor.RotateX(AXIAL_TILTS["jupiter"])
* Rendering: A striped orange-brown pv.Sphere (color="orange" with zonal bands via cylindrical texture mapping, e.g., JunoCam composites; opacity=1.0), featuring a persistent red oval for the Great Red Spot (procedural ellipse mesh). Faint dusty rings (ring_actors) orbit in the equatorial plane, scaled from Amalthea/Thebe ejecta
* Orbital Motion: Real-time update in update(): set_body_position_from_elements("jupiter", self.jupiter_actor, orbital_elements["jupiter"], target_jd). Heliocentric km-to-scene scaling (km_to_scene) highlights its near-circular path, with barycentric Sun wobble influence
<img width="1351" height="657" alt="image" src="https://github.com/user-attachments/assets/0dfbb754-8045-4564-87dc-dfd6c527ce1a" />

* Rotation: Ultra-fast prograde spin (actor.RotateZ(deg % 360.0)), equatorial bulge simulated by ellipsoidal mesh deformation for oblateness (~6% flattening).
* Moons: 95 bodies in MOON_SCALE_FACTORS["jupiter"] (e.g., Io at 421,800 km, e=0.0041; tiny Valetudo at 18.7M km), updated in moon loop with tidal locking (actor.RotateZ(v_deg)) and LOD scaling (TINY_MOON_THRESHOLD = 150,000 km). Inner Galileans (1–8: Metis to Callisto) prograde; outer irregulars retrograde
<img width="1351" height="662" alt="image" src="https://github.com/user-attachments/assets/319af0b4-53e3-4dde-83af-1ec627b3c697" />
Focus on Jupiter: Press j to zoom (set_planet_focus('jupiter', self.jupiter_actor)), then 1–95 for moons (e.g., 6=Ganymede; multi-digit buffered; min distance: ~JUPITER_RADIUS * 20 units).

SATURN
Saturn, the jewel of the solar system, graces 9.58 AU from the Sun as a pale yellow gas giant, its serene beauty defined by an extensive ring system—composed of billions of icy particles, rocky debris, and dust, spanning 282,000 km wide but mere meters thick—making it the most spectacular in our cosmic backyard. Beneath the hazy ammonia-methane atmosphere lie hydrogen-helium layers hiding a rocky core, with winds whipping up to 1,800 km/h and hexagonal storms at its north pole. Temperatures hover at -140°C (-220°F) in the clouds, warming to 11,700 K at the core. Home to 145+ moons (83 modeled here), including smog-shrouded Titan (larger than Mercury, with lakes of liquid methane) and geyser-spewing Enceladus (hinting at subsurface ocean habitability). NASA's Voyager (1980–1981) and Cassini-Huygens (2004–2017) missions revealed ring "spokes," moon shepherds like Prometheus, and Titan's eerie Earth-like dunes, cementing Saturn's role in reshaping our understanding of icy worlds.
In this simulation, Saturn captivates with its photorealistic rings (multi-layered: C, B, Cassini Division, A), 83 orbiting moons, and oblate spheroid form, powered by Keplerian orbits to depict its leisurely revolution and the intricate shepherding dynamics.
<img width="1361" height="658" alt="image" src="https://github.com/user-attachments/assets/d8efc058-7f39-4281-b0dc-00d1fed2d8d9" />
* Radius: 58,232 km (scaled to SATURN_RADIUS = REAL_SATURN_RADIUS_KM * SIZEKM_TO_SCENE for majestic scale)
* Gravitational Parameter (GM): 3.793 × 10⁷ km³/s² (GM_SATURN), anchoring its vast ring and moon ensembles
* Orbital Distance: 9.58 AU (SATURN_AU), generating SATURN_ORBIT_RADIUS = SATURN_AU * AU_SCALE
* Rotation Period: 10.656 hours (sidereal; ROTATION_PERIOD["saturn"] = 10.656 * 3600 seconds), yielding ~0.034° per simulated second (ROTATION_DEG_PER_SIMSEC["saturn"])—causing 10% equatorial bulge
<img width="1337" height="638" alt="image" src="https://github.com/user-attachments/assets/d946c1a4-3ce1-4f60-94da-401d389b4f94" />

* Axial Tilt: 26.73° (AXIAL_TILTS["saturn"]), promoting seasonal ring tilts, via actor.RotateX(AXIAL_TILTS["saturn"]) and ring co-rotation
* Rendering: A buttery-yellow pv.Sphere (color="gold" or "khaki" with banded haze textures from Cassini; opacity=0.9), oblate via ellipsoidal mesh (flattening ~10%). Iconic rings (ring_actors) as nested toroidal meshes (C: 74,658–92,000 km; B: 92,000–117,580 km; Cassini gap; A: 122,170–136,775 km), textured with icy particle noise for spokes and braids
<img width="1350" height="655" alt="image" src="https://github.com/user-attachments/assets/40b8e7da-2e5d-4c38-9ba5-f31be52a5563" />

* Orbital Motion: Dynamic positioning in update(): set_body_position_from_elements("saturn", self.saturn_actor, orbital_elements["saturn"], target_jd). Scene-scaled heliocentric coords (km_to_scene) emphasize its steady, near-circular trek.
<img width="1355" height="659" alt="image" src="https://github.com/user-attachments/assets/b1946652-d769-4d97-8d57-d265bf43970a" />

* Rotation: Swift prograde whirl (actor.RotateZ(deg % 360.0)), with rings rigidly attached (ractor.RotateX(AXIAL_TILTS["saturn"]) and position-synced)
* Rings and Moons: Multi-ring system follows Saturn (for ractor in self.ring_actors: ractor.SetPosition(*self.saturn_actor.GetPosition())), with 83 moons in MOON_SCALE_FACTORS["saturn"] (e.g., Mimas at 186,000 km, e=0.020; Titan at 1,221,900 km, e=0.029). Moon loop handles tidal locking (actor.RotateZ(v_deg)) and scaling (TINY_MOON_THRESHOLD for inners like Pan); outer irregulars retrograde
<img width="1354" height="661" alt="image" src="https://github.com/user-attachments/assets/e9d0f4bd-a29b-465f-b760-aba670fd51b9" />

* Focus on Saturn: Press s to immerse (set_planet_focus('saturn', self.saturn_actor)), then 1–83 for moons (e.g., 15=Enceladus, 20=Titan; buffered numerics; min distance: ~SATURN_RADIUS * 20 units.
URANUS
Uranus, the sideways ice giant and seventh wanderer, lurks at 19.22 AU from the Sun, its pale cyan hue from methane-absorbed red light in a frigid atmosphere of hydrogen (83%), helium (15%), and methane (2%)—the coldest in the solar system at -224°C (-371°F) in the clouds. Famously tilted 98° on its axis (likely from a colossal ancient impact), it "rolls" around the Sun like a tipped bowling ball, causing extreme 42-year seasons where poles bask in sunlight while equators freeze in darkness. Beneath the hazy layers lies a slushy mantle of water, ammonia, and methane ices encasing a rocky core, with faint, dusty rings (13 known) and 27 moons—from tiny Miranda (geologically fractured) to shepherding Cordelia. Voyager 2's flyby (1986)—the only visit—revealed a bland, featureless disk hiding supersonic winds (up to 900 km/h) and a magnetic field oddly offset from its spin axis, unraveling enigmas of this "forgotten" world now eyed for future missions like NASA's proposed Uranus Orbiter.
In this simulation, Uranus intrigues with its bizarre tilt (rings/moons co-rotating sideways), 27 dynamically orbiting satellites, and ethereal rings, harnessed by Keplerian mechanics to dramatize its ponderous, rolling orbit and seasonal extremes.
<img width="1349" height="656" alt="image" src="https://github.com/user-attachments/assets/12fdf5b5-fc42-432b-beca-284ed8c5c7ac" />

* Radius: 25,559 km (scaled to URANUS_RADIUS = REAL_URANUS_RADIUS_KM * SIZEKM_TO_SCENE for icy bulk)
* Gravitational Parameter (GM): 5.794 × 10⁶ km³/s² (GM_URANUS), stabilizing its tilted ring-moon system
* Orbital Distance: 19.22 AU (URANUS_AU), crafting URANUS_ORBIT_RADIUS = URANUS_AU * AU_SCALE
* Rotation Period: -17.24 hours (retrograde; ROTATION_PERIOD["uranus"] = -17.24 * 3600 seconds), ~ -0.021° per simulated second (ROTATION_DEG_PER_SIMSEC["uranus"])—negative for backward spin
<img width="1348" height="655" alt="image" src="https://github.com/user-attachments/assets/b1ce043d-8fe0-41ae-aefc-7855a46cbabe" />

* Axial Tilt: 97.77° (AXIAL_TILTS["uranus"]), near-perpendicular for "sideways" motion, via actor.RotateX(AXIAL_TILTS["uranus"]) and ring co-tilt
* Rendering: A serene blue-green pv.Sphere (color="cyan" with hazy methane bands via subtle noise textures; opacity=0.8), oblate (~2% flattening) to evoke Voyager's blandness. Faint epsilon-ring system (uranus_ring_actors) as thin, clumpy tori (e.g., epsilon at ~51,000 km), textured for dust/ice variabilit
* Orbital Motion: Updated in update():set_body_position_from_elements("uranus", self.uranus_actor, orbital_elements["uranus"], target_jd)Scaled km-to-scene (km_to_scene) underscores its glacial 84-year lap
<img width="1341" height="647" alt="image" src="https://github.com/user-attachments/assets/480589da-61cb-44bb-a4e2-4ba8e482d71c" />

* Rotation: Retrograde axial roll (actor.RotateZ(-deg % 360.0) for negative period), with rings synced (for ractor in self.uranus_ring_actors: ractor.RotateX(AXIAL_TILTS["uranus"]))
* Rings and Moons: 13 dusty rings position-locked to Uranus (ractor.SetPosition(*self.uranus_actor.GetPosition())), with 27 moons in MOON_SCALE_FACTORS["uranus"] (e.g., Miranda at 129,900 km, e=0.0013; outer Sycorax at 12.2M km, e=0.522). Moon updates include tidal locking (actor.RotateZ(v_deg)) and retrograde outer irregulars; URANUS_TINY_THRESHOLD = URANUS_RADIUS * 0.01 culls for perf
<img width="1349" height="656" alt="image" src="https://github.com/user-attachments/assets/f28a66f3-0cd0-499b-a02a-dc6bf35bc964" />

* Focus on Uranus: Press u to tilt into view (set_planet_focus('uranus', self.uranus_actor)), then 1–27 for moons (e.g., 12=Miranda, 20=Sycorax; multi-digit; min distance: ~URANUS_RADIUS * 20 units)
NEPTUNE
Neptune, the azure ice giant and outermost major planet, dwells at 30.07 AU from the Sun, its deep blue mantle forged by methane ice crystals scattering sunlight in a turbulent hydrogen-helium atmosphere laced with hydrogen sulfide for its stormy temperament—home to the solar system's fiercest winds (up to 2,100 km/h) and the Great Dark Spot, a transient anticyclone rivaling Earth in size. Frigid at -201°C (-330°F) in the clouds, it harbors a diamond rain interior and a rocky core under supercritical fluid layers, with faint, clumpy rings (five arcs: Adams to Galle) and 16 moons (14 modeled here), led by retrograde Triton—likely a captured Kuiper Belt object spewing nitrogen geysers from its icy crust. Voyager 2's solitary flyby (1989) unveiled these tempests and a bizarre magnetic field tilted 47° from its spin axis, while James Webb Space Telescope glimpses (2022–present) reveal evolving spots and ring bows, positioning Neptune as a dynamic frontier for ocean world studies.
In this simulation, Neptune enchants with its 14 tidally diverse moons, subtle ring arcs (if extended), and windy banded globe, driven by Keplerian orbits to evoke its languid 165-year voyage and Triton's dramatic backward tumble.
<img width="1347" height="650" alt="image" src="https://github.com/user-attachments/assets/c30c6e0e-8408-412f-9c4b-2caf40a1d427" />
* Radius: 24,622 km (scaled to NEPTUNE_RADIUS = REAL_NEPTUNE_RADIUS_KM * SIZEKM_TO_SCENE for distant allure)
* Gravitational Parameter (GM): 6.836 × 10⁶ km³/s² (GM_NEPTUNE), tethering its irregular satellites
* Orbital Distance: 30.07 AU (NEPTUNE_AU), forming NEPTUNE_ORBIT_RADIUS = NEPTUNE_AU * AU_SCALE
<img width="1346" height="652" alt="image" src="https://github.com/user-attachments/assets/805a6bb6-cbd2-471a-85c0-4d510a69d3be" />

* Rotation Period: 16.11 hours (sidereal; ROTATION_PERIOD["neptune"] = 16.11 * 3600 seconds), ~0.022° per simulated second (ROTATION_DEG_PER_SIMSEC["neptune"]) for brisk day
* Axial Tilt: 28.32° (AXIAL_TILTS["neptune"]), fostering asymmetric seasons, via actor.RotateX(AXIAL_TILTS["neptune"])
* Rendering: A vivid indigo pv.Sphere (color="blue" with zonal cloud streaks via banded textures from Voyager; opacity=0.85), subtly oblate (~1% flattening). Faint ring system (extendable via uranus_ring_actors pattern) as arcuate dust bands (e.g., Adams at 63,000 km), with procedural clumping for bow shocks
* Orbital Motion: Live refresh in update(): set_body_position_from_elements("neptune", self.neptune_actor, orbital_elements["neptune"], target_jd)Km-to-scene conversion (km_to_scene) captures its elliptical wanderings
<img width="1351" height="648" alt="image" src="https://github.com/user-attachments/assets/c7ef5a49-508e-4b0d-8ad7-d2582d658bbb" />

* Rotation: Prograde axial twirl (actor.RotateZ(deg % 360.0)), evoking spot drifts
* Moons: 14 in MOON_SCALE_FACTORS["neptune"] (e.g., Naiad at 48,224 km, e=0.0047; Triton at 354,759 km, i=156.8° retrograde), processed in moon loop with locking (actor.RotateZ(v_deg)) and scaling (NEPTUNE_TINY_THRESHOLD = NEPTUNE_RADIUS * 0.01 for inners like Proteus). Outer irregulars (e.g., Neso at 49.2M km) highlight capture origins
<img width="1351" height="655" alt="image" src="https://github.com/user-attachments/assets/de5c7546-3eb0-4b27-97d4-76f04620623a" />

* Focus on Neptune: Press n to plunge in (set_planet_focus('neptune', self.neptune_actor)), then 1–14 for moons (e.g., 8=Triton; buffered; min distance: ~NEPTUNE_RADIUS * 20 units)
Encke (2P/Encke) 
Comet Encke is the fastest short-period comet in our solar system, completing an orbit every 3.3 years — the shortest known period for any comet. Discovered in 1786 by Pierre Méchain and rediscovered multiple times (hence its multiple historical names), it’s officially designated 2P/Encke and belongs to the rare Encke-type comet family inside Jupiter’s orbit.
Unlike dramatic long-period visitors like Hale-Bopp, Encke has been repeatedly stripped by solar heating over thousands of passes, leaving it with very low activity and no visible dust tail in most apparitions. What remains is a faint ion tail and a small coma, making it a challenging naked-eye object (best magnitude ~3–6). It’s considered the parent of the Taurid meteor stream (responsible for November’s Taurids fireballs) and may be a fragment of a much larger ancient comet.
In this simulation, Encke is fully dynamic with real JPL Horizons osculating elements (epoch Nov 2025), a volumetric coma & tail, and distance-based sublimation — it only activates inside ~2.2 AU (adjustable via SUBLIMATION_DISTANCE).
<img width="1350" height="649" alt="image" src="https://github.com/user-attachments/assets/a19b626b-3356-4cc2-8509-23e6a3ab1db7" />
* Semi-major axis: 2.215 AU
* Eccentricity: 0.84833 → highly elongated orbit
* Inclination: 11.76°
* Perihelion distance: ~0.33 AU (very close to the Sun!)
* Orbital period: 3.3 years
* Nucleus radius: ~2.0 km (very small, depleted)
* Epoch: JD 2460990.5 (Nov 11, 2025) — elements directly from JPL Horizons
* Color in sim: Yellow/orange ("enscke": "YlOrBr" colormap) — reflects dusty, low-gas composition
* Nucleus: Tiny glowing point (scaled up for visibility) using nucleus_actor
<img width="1346" height="640" alt="image" src="https://github.com/user-attachments/assets/81f4bd2b-324a-4ec8-a8cd-013933376784" />
* Coma & Tail: Fully volumetric using make_volumetric_comet_body()
* Activates only when r_km < SUBLIMATION_DISTANCE (currently 10 AU, but realistic ~2.2 AU)
* Tail points anti-solar and aligns with velocity vector
* Turbulence, vortex, and flicker effects for realism
* Custom colormap "YlOrBr" → warm yellowish appearance matching its dust-rich nature
* Halo/Glow: Cyan-white coma halo that brightens near perihelion
* Motion: Smoothed Keplerian propagation using kepler_to_state() with mu = GM_SUN
* Press l → Focus on Comet Encke instantly
Halley (1P/Halley)
1P/Halley is the archetypal periodic comet and the only one visible to the naked eye that can appear twice in a human lifetime. First recorded by humans in 240 BC, it has been observed on every return since, including spectacular showings in 1066 (Bayeux Tapestry) and 1910. Its most recent visit was in 1986 (studied by Giotto, Vega, and Suisei probes), and it will return next in mid-2061, reaching perihelion on July 28, 2061.
Halley is a classic long-period comet originating from the Oort Cloud, highly inclined and retrograde (162°), traveling opposite to the planets. It produces both a dust tail (curved, yellowish) and a straight blue ion tail, often stretching tens of millions of kilometers — exactly what the volumetric engine in this simulation was built to showcase.
Halley close-up
<img width="1292" height="582" alt="image" src="https://github.com/user-attachments/assets/3fa98ced-7473-4903-8990-263d485cc763" />
* Semi-major axis: 17.84 AU
* Eccentricity: 0.9671 → extremely elongated orbit
* Inclination: 162.26° (retrograde!)
* Perihelion distance: ~0.586 AU
* Orbital period: 75.32 years (next return: 2061)
* Nucleus size: ~15 × 8 × 8 km (peanut-shaped, very dark — albedo ~0.04)
* Nucleus radius used: 5.5 km (effective for rendering)
* Epoch: JD 2460990.5 (Nov 11, 2025) — exact JPL Horizons elements
* Color in sim: Greenish ("Greens" colormap) — reflects strong CN and C₂ emissions seen in 1986
<img width="1342" height="609" alt="image" src="https://github.com/user-attachments/assets/cad983b8-3264-473b-96b0-482ef645c9f3" />
* Nucleus: Dark, potato-shaped core (scaled up for visibility)
* Coma & Tail: Full volumetric treatment via make_volumetric_comet_body()
* Activates inside SUBLIMATION_DISTANCE (10 AU in code, realistic ~4–5 AU for Halley)
* Gorgeous dual tail structure: dust (curved) + ion (straight) naturally emerges from turbulence and velocity alignment
* Green colormap perfectly matches Halley’s famous cyan-green coma
* Tail length scales with activity — can exceed 0.5 AU near perihelion
* Retrograde Motion: Orbits opposite to planets — watch it race against the ecliptic flow!
* Press z → Instantly focus on Comet Halley

C/1995 O1 (Hale-Bopp) 
Hale-Bopp is widely regarded as the Great Comet of the 20th century and one of the brightest comets ever witnessed in human history. Independently discovered on July 23, 1995 by Alan Hale and Thomas Bopp while it was still beyond Jupiter’s orbit (7.1 AU from the Sun), it remained visible to the naked eye for a record-breaking 18 months — longer than any other comet in recorded history. At its peak in March–April 1997 it reached magnitude -1, outshining every star except Sirius, and displayed an enormous dual tail system visible even from light-polluted cities. It was photographed billions of times and sparked worldwide fascination (and unfortunately the Heaven’s Gate tragedy). Unlike Halley, Hale-Bopp will not return for thousands of years.
Hale-Bopp is a true Oort Cloud giant with an estimated nucleus diameter of 40–60 km (up to 10× larger than Halley), extremely high dust production, and activity that began while still beyond Saturn. Its near-perpendicular orbit (89.3°) sent it plunging almost straight through the planetary plane, producing one of the most spectacular dust tails ever recorded (over 40° long) alongside a narrow, straight blue ion tail — the perfect showcase for the volumetric comet engine.

Hale-Bopp close-up
<img width="1350" height="655" alt="image" src="https://github.com/user-attachments/assets/5be9991d-bbab-4fac-bb68-a76021eea0ff" />

* Semi-major axis: ~177.43 AU (extremely long-period comet)
* Eccentricity: 0.99498 → essentially parabolic after 1997 perturbations (will never return on human timescales)
* Inclination: 89.29° (almost perfectly perpendicular to the ecliptic)
* Perihelion distance: 0.914 AU (passed perihelion on April 1, 1997)
* Orbital period: ~2520 years inbound → ~4200 years outbound after planetary encounters
* Nucleus size: ~40–60 km diameter (one of the largest known cometary nuclei)
* Nucleus radius used: 30 km (effective for rendering)
* Epoch: JD 2460990.5 (Nov 11, 2025) — exact JPL Horizons elements
* Color in sim: Deep blue-to-cyan ("Blues" colormap) — captures the bright white coma and stunning pale blue ion tail seen in 1997
<img width="1345" height="652" alt="image" src="https://github.com/user-attachments/assets/a7e5aad3-59b1-4eef-90bb-0ed11461e2f2" />

* Nucleus: Massive, hyperactive even at 7+ AU (visible naked-eye while beyond Jupiter!)
* Coma & Tail: Full volumetric treatment via make_volumetric_comet_body()
* Activates inside SUBLIMATION_DISTANCE (realistic ~7–8 AU — it was already brilliant at 7 AU pre-perihelion)
* Legendary dual tail structure: enormous curved white-yellow dust tail (up to 40°+ long) + narrow straight blue ion tail — naturally emerges from turbulence and velocity alignment
* "Blues" colormap perfectly recreates the ghostly white coma fading into the pale blue plasma tail
<img width="1343" height="649" alt="image" src="https://github.com/user-attachments/assets/abc666d4-1422-44aa-8d94-4e4d85de87e9" />

* Tail length scales with activity — reached over 1 AU in reality, easily >0.8 AU in sim near perihelion
* Near-polar orbit — watch it dive almost straight down through the ecliptic like a cosmic spear!
* Press x → Instantly focus on Comet Hale-Bopp

C/2011 W3 (Lovejoy)
Lovejoy is one of the most dramatic and photogenic comets of the 21st century — the only Kreutz sungrazer in modern history to survive a direct plunge through the solar corona. Discovered on 27 November 2011 by Australian amateur Terry Lovejoy when it was still a faint 12th-magnitude object, it brightened explosively as it approached the Sun and, on 16 December 2011, passed just 140 000 km (0.009 AU) above the solar surface at over 600 km/s. Expected to disintegrate like most Kreutz fragments, it instead emerged intact on the other side, regenerating a brilliant glowing green coma and a spectacular long tail within hours. For weeks afterward it put on a breathtaking pre-dawn show for the entire Southern Hemisphere, earning the nicknames “Christmas Comet” and “Great Southern Comet.”
Lovejoy belongs to the famous Kreutz sungrazer family — fragments of a giant comet that broke apart centuries ago. Its extreme perihelion vaporized vast amounts of material, releasing huge quantities of diatomic carbon (C₂) and cyanogen that produced the intense emerald-green color that made it instantly recognizable in thousands of iconic photographs.
Lovejoy close-up
<img width="1354" height="648" alt="image" src="https://github.com/user-attachments/assets/91e3db1e-bbd8-4221-bbf7-a1f59ddf8dae" />


* Semi-major axis: ~393 AU (very long-period Oort Cloud comet)
* Eccentricity: 0.998 → essentially parabolic, will be ejected from the Solar System after this pass
* Inclination: 80.3° (highly inclined)
* Perihelion distance: 0.009 AU (140 000 km above the Sun’s surface — survived by sheer luck!)
* Orbital period: hundreds of thousands of years inbound → hyperbolic outbound
* Nucleus size: estimated 0.5–2 km (typical small Kreutz fragment; most of it vaporized)
* Nucleus radius used: 10 km (scaled up for visibility and volumetric rendering)
* Epoch: JD 2460990.5 (Nov 11, 2025) — exact JPL Horizons elements
* Color in sim: Intense emerald green ("Greens" colormap) — perfectly matches the real C₂ swan-band emission that gave Lovejoy its signature glowing green comaimage
* Nucleus: Tiny surviving fragment that somehow lived through million-degree temperatures
* Coma & Tail: Full volumetric treatment via make_volumetric_comet_body()
<img width="1343" height="637" alt="image" src="https://github.com/user-attachments/assets/35741708-8f8e-40ec-8a3b-8fbf7e871763" />

* Activates far out (~10 AU) but goes absolutely nuclear inside ~2 AU, with explosive activity right at perihelion
* Gorgeous glowing emerald coma + long straight ion tail + curved dust tail — the volumetric engine nails the vivid green color and turbulent post-perihelion structure
* "Greens" colormap is a perfect reproduction of the diatomic-carbon green seen in real images
* Tail length scales with activity — post-perihelion tail stretched tens of millions of km, easily >0.6 AU in sim
* Watch it scream in from the Oort Cloud, graze the Sun at 600 km/s, and get flung out of the Solar System forever!
* Press d → Instantly focus on Comet Lovejoy


Rendering a realistic comet body is a very heavy task. At first I tried creating real dust and ion tails using millions of tiny dots exactly like real comets, but it lagged terribly. I experimented many different ways, but on my low-end laptop it was impossible. Eventually I switched to volumetric rendering: I create a rectangular block, make the comet head and tail regions denser, then zero-out everything else. This runs, but it’s still quite laggy and hurts the smoothness of the entire simulation.  
To make it run smoothly on lower-end machines you can reduce the grid resolution (COMET_GRID_SIZE from 128 → 64 or even 48) and lower the gaussian_filter sigma — you’ll lose some fine detail but gain huge performance and the comet will still look beautiful.  
I know how to make it look 100 % like real-world photos (turbulence layers, synchrotron ion tail rays, anti-tail, striae, etc.), but I simply don’t have the hardware to test and run it in real time.  
There are also many more things I wanted to add: dozens of parabolic/hypothetical comets, full Kuiper belt cloud, hundreds more moons with real NASA textures, dwarf planets textures, asteroid families, etc. — the script will never be “complete”. Feel free to use this project as a reference for study or further development.

midhun10112003@gmail.com → contact me anytime if you have questions, need guidance, or want to collaborate.  
This is not an academic or commercial project — I just do it in my free time, may be  I’m  obsessed with the universe and celestial mechanics ♡

by
Midhun V P
