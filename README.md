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

