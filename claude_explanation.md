# Challenge
Self-driving car design is a broad topic, so here’s a compact overview of how these systems are typically architected.

Core pipeline

Sensing: cameras (rich semantics, cheap), radar (velocity, works in bad weather), lidar (precise 3D geometry), plus GPS/IMU and wheel odometry. Tesla bets on camera-only; Waymo uses a full multi-sensor suite.
Perception: object detection and tracking (cars, pedestrians, cyclists), lane and drivable-area segmentation, traffic light/sign recognition, and increasingly end-to-end 3D occupancy or bird’s-eye-view (BEV) models.
Localization and mapping: pinpointing the vehicle to centimeter level via HD maps plus sensor matching (SLAM, lidar map matching), or map-light approaches relying on live perception.
Prediction: forecasting where other agents will go over the next several seconds, usually multimodal (several plausible futures with probabilities).
Planning: a route planner picks roads, a behavior planner decides maneuvers (merge, yield, overtake), and a motion planner generates a smooth, safe, comfortable trajectory.
Control: PID/MPC controllers turn the trajectory into steering, throttle, and braking commands.
Safety layer: redundancy, fallbacks (minimal-risk maneuver), and monitoring that can override the main stack.

Two main design philosophies

Modular: separate perception, prediction, planning. Easier to debug, validate, and certify, but errors compound across interfaces.
End-to-end: a neural network maps sensors to trajectories or controls. Scales better with data and handles long-tail cases more naturally, but is harder to interpret and verify. Many real systems are hybrids.

Key challenges

The long tail of rare scenarios
Verification: proving safety statistically (billions of miles via simulation plus real-world testing)
Compute and power budgets on-vehicle, with real-time latency limits
Redundancy in brakes, steering, power, and compute
Regulation and liability, including SAE levels (L2 driver assist through L4/L5 autonomy)

Supporting infrastructure: data engine for mining hard cases, auto-labeling, large-scale simulation, OTA updates, and fleet monitoring.


# self driving vehicle
For vehicles that carry things in production places like factories and warehouses, the more common industry terms are:

AGV (Automated Guided Vehicle): follows fixed paths such as magnetic tape, wires, or markers
AMR (Autonomous Mobile Robot): navigates freely using sensors and maps, avoiding obstacles on its own
Driverless vehicle or autonomous vehicle: general terms, also used for trucks in mines and ports
Automated forklift or autonomous forklift: for pallet handling
Autonomous haul truck: for mining and quarry sites

“Self-driving vehicle” isn’t wrong, but it sounds more like road cars. In industrial settings, AGV and AMR are the standard terms.
There's no single official count, but in industrial settings they fall into about 7 families, mainly separated by how they navigate.

| # | Type | Navigation | Typical use |
|---|------|-----------|-------------|
| 1 | **AGV** (Automated Guided Vehicle) | Fixed path: magnetic tape, embedded wire, QR/RFID markers, laser reflectors | Repetitive factory routes, assembly lines |
| 2 | **AMR** (Autonomous Mobile Robot) | Free navigation: SLAM, lidar and camera, dynamic obstacle avoidance | Warehouses, flexible factory floors |
| 3 | **Autonomous forklift / pallet truck** | AGV-style or AMR-style, plus lift mast | Pallet loading, racking, trailer unloading |
| 4 | **Autonomous tugger / tow tractor** | Mostly AGV-style, sometimes AMR | Towing cart trains, line-side supply |
| 5 | **Rail-guided and overhead systems** (RGV, OHT) | Physical rails or ceiling tracks | Semiconductor fabs, automated storage, sorting |
| 6 | **Shuttle / goods-to-person robots** | Grid or rack navigation, centrally controlled | Dense warehouse storage, e-commerce fulfillment |
| 7 | **Heavy outdoor driverless trucks** | GNSS, lidar, radar, HD site maps | Mining haul trucks, port container carriers, yard trucks |

**Common skeleton (all types share it)**

1. **Chassis**: drive and steering (differential, omni/mecanum, or Ackermann), battery or charging system
2. **Sensors**: encoders and IMU, plus lidar, cameras, or markers/reader depending on type
3. **Localization**: where am I (marker-based for AGV, SLAM or map-matching for AMR, GNSS for outdoor)
4. **Planning**: route (global) and obstacle avoidance (local); fixed in AGV, dynamic in AMR
5. **Control**: motor, speed, and steering control
6. **Safety**: safety-rated lidar scanners, bumpers, e-stop, speed zones (ISO 3691-4 is the main standard)
7. **Fleet management**: task assignment, traffic control, charging, integration with WMS/MES

**What really differs between types**
- AGV is simple and reliable but inflexible.
- AMR is flexible but needs more compute and tuning.
- Forklifts and tuggers are mostly AGV/AMR with a payload mechanism added.
- Outdoor heavy vehicles add GNSS, weather resilience, and much higher safety stakes.

# for factory use
Understood: you're designing a driverless vehicle for factory use. Here are examples grouped by the job they do, since the job shapes the design.

| Factory job | Vehicle example | Design notes |
|---|---|---|
| **Line-side part delivery** (small parts, totes) | Small AMRs with a top module, such as MiR100/250 or OTTO 100 | Low payload, free navigation, shared space with people |
| **Heavy load transport** (engines, battery packs, 500 kg to 1.5 t) | MiR600/1350, KUKA KMP series | Stronger chassis, longer stopping distance, stricter safety zones |
| **Pallet moving and stacking** | Autonomous forklifts and pallet trucks (Toyota, Jungheinrich, Linde, Seegrid) | Lift mast, pallet detection, load-handling safety |
| **Assembly line carrying** (car body or chassis moves station to station) | Underbody or "marriage" AGVs | Fixed path, high positioning accuracy (millimeters), synchronized with the line |
| **Towing carts** (milk-run supply loops) | Tugger AGVs (tow tractors) | Predictable routes, cart coupling mechanism |
| **Clean-room or fab transport** | Overhead hoist transport (OHT) and rail-guided vehicles | Ceiling or rail guided, very high reliability |
| **Mobile manipulation** (machine tending) | AMR with a robot arm (Type C) | Arm and base must be safety-validated together |

**Typical design choices**
- **Predictable route, high volume:** AGV (tape, QR, or laser reflectors) is cheaper and simpler.
- **Changing layout, mixed human traffic:** AMR with SLAM, lidar, and fleet software.
- **Both are checked against ISO 3691-4** for safety.

# some public designs
Complete public design files (CAD, schematics, firmware) for commercial factory vehicles are proprietary. What is public falls into two groups: product datasheets, which show real design targets, and academic or open-source designs, which show how one is built.

**Commercial datasheets (real specs to benchmark against)**

- **MiR250 specifications.** This is a small AMR measuring 800 x 580 x 300 mm that weighs 83 kg, carries up to 250 kg, and reaches 2.0 m/s (this is the 2020 edition). Newer editions are the 2023 and 2025 versions of the same spec sheet. PDF: https://www.hartfiel.com/wp-content/uploads/2024/04/MiR250-Specs.pdf
- **MiR250 datasheet (older, 2021).** It lists the chassis, wheels, payload limits, and runtime, including 13 hours of operation at full load. PDF: https://howtorobot.com/sites/default/files/2021-02/MiR250%20_%20Mobile%20Industrial%20Robots.pdf
- **IMR250 (MiR250 base plus a FANUC arm).** This is a mobile manipulator example, with base repeatability of ±85 mm without a dock and ±4.25 mm with a dock. PDF: https://howtorobot.com/sites/default/files/2026-05/IMR250%20Specifications%201.pdf

**Academic and open designs (how to build one)**

- **GOFR, an autonomous shop floor robot with ROS.** The paper covers the hardware architecture, instrumentation, communication, and control systems of a robot for material transport on manufacturing shop floors. PDF: https://peer.asee.org/design-of-an-autonomous-shop-floor-robot-gofr-with-ros.pdf
- **ROS-based AMR for intralogistics in manufacturing (JAMRIS, 2026, open access).** It combines a mechanical design, sensor fusion, and Hybrid A* path planning, and was verified in both simulation and real settings. Page: https://reference-global.com/article/10.14313/jamris-2026-010
- **Low-cost ROS2 carrier robot for structured factories.** It targets small and medium manufacturers that need an affordable solution. Page: https://journal.ump.edu.my/index.php/isse/article/view/14255
- **Development and testing of an AMR for material handling (thesis).** It covers the design phase, SLAM, ROS2 Control, Nav2, and ArUco tag detection. PDF: https://elibrary.tucl.edu.np/bitstreams/7cc70310-cbc0-414b-a9c5-da2568fd8124/download

I only saw search previews, not the full files, so check each one for relevance. The MiR sheets are best for choosing target specs, and the ROS papers are best for software architecture. For safety you will still need the ISO 3691-4 whitepapers from earlier.

# Types
Driverless vehicles exist in many domains. Grouped by where they operate:

| Domain | Kinds | Examples |
|---|---|---|
| **Factory and warehouse (your focus)** | AGV, AMR, autonomous forklift, tugger, rail/overhead (OHT), shuttle/goods-to-person robots | MiR, OTTO, Seegrid, Kiva-style robots |
| **Outdoor industrial** | Mining haul trucks, port container carriers (AGV/ALV), yard trucks, construction machines | Komatsu FrontRunner, Caterpillar Command, Kalmar, Konecranes |
| **Road vehicles** | Robotaxis, autonomous trucks (highway), shuttles/buses, delivery vans | Waymo, Aurora, Navya/EasyMile |
| **Last-mile delivery** | Sidewalk robots, small delivery pods | Starship, Nuro |
| **Agriculture** | Driverless tractors, harvesters, orchard and spraying robots | John Deere autonomous tractor |
| **Rail** | Driverless metro, automated freight trains, autonomous haul trains | Paris Line 14, Rio Tinto AutoHaul |
| **Air** | Cargo and delivery drones, autonomous aircraft/eVTOL | Zipline, Wing |
| **Sea** | Autonomous ships, underwater vehicles (AUV), surface drones | Yara Birkeland, research AUVs |
| **Specialty** | Airport baggage tractors, hospital delivery robots, cleaning robots, military UGVs | Aethon TUG |

**Two ways to classify them**

- **By autonomy level.** Road vehicles use SAE L0 to L5, and L4 means fully driverless in a defined area. Most factory vehicles are effectively L4 within a controlled site.
- **By navigation.** The three main types are fixed path (tape, wire, rails), reflector or marker-based, and free navigation (SLAM, GNSS, HD maps).

The factory kinds (AGV, AMR, forklift, tugger, rail, shuttle) are the 7 families in your spreadsheet. The other domains are mostly useful for borrowing ideas, since factory vehicles usually have simpler environments and tighter safety rules.


# Types of AI module
AI fits into a factory vehicle as several separate modules. The key rule is that AI handles perception and optimization, while a certified non-AI safety layer handles stopping.

| Module | What AI does | Typical methods | Needed for |
|---|---|---|---|
| **Object detection** | Finds people, forklifts, pallets, carts | CNN detectors (YOLO family), transformer detectors | AMR, forklift |
| **Semantic segmentation** | Labels floor, aisle, obstacle, no-go zone | Segmentation networks (U-Net, DeepLab) | AMR |
| **Pallet and cart detection** | Locates pallet pockets, cart legs, docking targets | Depth camera plus a detection or pose model | Forklift, hook/lift top modules |
| **Localization and mapping** | Position estimate and map building | SLAM (lidar or visual), learned loop closure | AMR (AGV uses markers instead) |
| **Human and motion prediction** | Predicts where workers and other vehicles will go | Trajectory prediction networks | AMR in shared spaces |
| **Path planning** | Chooses routes and avoids obstacles | Classical planners (Nav2, Hybrid A*), sometimes reinforcement learning | AMR |
| **Fleet optimization** | Task assignment, traffic control, charging schedule | Optimization solvers, learned scheduling | Any multi-vehicle site |
| **Predictive maintenance** | Spots battery, motor, and wheel wear from sensor data | Anomaly detection, time-series models | Any |
| **Vision-language / end-to-end models** | Understands instructions or scenes ("go to station 4") | VLMs, end-to-end policies | Research stage, not yet standard on factory floors |

**What to use for a first design**
- **Start with:** an object detector, a segmentation model, and SLAM, running on an embedded GPU module (NVIDIA Jetson Orin class) under ROS 2.
- **Keep classical:** path planning and motion control are usually still non-learning algorithms, because they are predictable and easy to validate.

**Safety limit**
- The stop-and-slow functions, meaning the safety lidar scanners, bumper, e-stop, and safety PLC, must stay as certified, deterministic hardware under ISO 3691-4 (PL d).
- AI-based detection can add to this layer, for example by slowing down earlier when it sees a person, but it should not replace it. A neural network is hard to certify and can fail in ways nobody predicted.

If you tell me whether you are building an AGV or an AMR and what it carries, I can list the specific sensors and models for each module.

# for AMR
Here is a concrete starting stack for an AMR, module by module.

**Sensors**

| Sensor | Role | Notes |
|---|---|---|
| 2 safety lidar scanners (2D, safety-rated) | Stop and slow zones | Certified hardware, not part of the AI |
| 1 navigation lidar (2D or 3D) | SLAM and localization | Can be the same scanners if the vendor allows |
| 1 to 2 depth cameras (RGB-D) | Detection and 3D obstacles | Front-facing, plus one covering the load area |
| Wheel encoders + IMU | Odometry | Fused with lidar for stable pose |
| Optional: ultrasonic or ToF sensors | Low obstacles, glass | Fills lidar blind spots |

**AI and software modules**

| Module | Recommended | Runs on |
|---|---|---|
| **Localization and mapping** | Lidar SLAM (Cartographer or slam_toolbox) with AMCL for localization | CPU |
| **Navigation** | ROS 2 Nav2 (global planner plus local controller) | CPU |
| **Object detection** | YOLO-family detector (small or medium model) trained on your factory images: people, forklifts, carts, pallets | GPU |
| **Segmentation** | Lightweight network (e.g. U-Net variants) for floor vs obstacle | GPU |
| **Docking and pallet or cart pose** | Depth camera plus a pose or marker model (AprilTag as a fallback) | GPU |
| **Human awareness** | Detector output feeds speed limits and path costs (slow near people) | GPU + planner |
| **Fleet and maintenance** | Fleet manager (open-source such as Open-RMF, or a vendor product) plus anomaly detection on battery and motor logs | Server |

**Compute**
- A Jetson Orin-class embedded GPU module for vision, plus a small industrial PC or the Jetson's CPU for SLAM and navigation.
- ROS 2 as the middleware, with the safety PLC kept on a separate, certified path.

**Safety split**
- Certified layer (non-AI): safety scanners, bumper, e-stop, safety PLC. These stop the vehicle.
- AI layer: detection, prediction, and planning. These make the vehicle slow down or reroute earlier, so the safety layer rarely has to trigger.

**Build order that works well**
1. Get SLAM and Nav2 driving reliably with lidar only.
2. Add the object detector and use it only to slow down near people.
3. Add docking or pallet detection for your top module.
4. Collect factory data and fine-tune the models.

To tune this, I need your payload and whether workers share the floor with the vehicle. Those two answers decide the sensor coverage and the speed limits.

# for AGV
An AGV needs much less AI than an AMR, because the path is fixed and the environment is controlled. Most of an AGV's reliability comes from simple, deterministic guidance.

**Guidance options (pick one)**

| Method | How it works | Pros and cons |
|---|---|---|
| **Magnetic tape** | Sensor follows tape on the floor | Cheap and simple, but the tape wears and path changes need re-taping |
| **QR or RFID markers** | Camera or reader localizes on floor tags | Flexible grid, accurate, very common in warehouses |
| **Laser reflectors** | Lidar triangulates on wall reflectors | High accuracy for large forklift AGVs, but needs reflector installation |
| **Embedded wire** | Induction coil follows a buried wire | Very robust, but costly to install and hard to change |

**Sensors**

| Sensor | Role |
|---|---|
| Guidance sensor (tape, tag reader, or lidar) | Path following |
| 1 to 2 safety lidar scanners (safety-rated) | Stop and slow zones |
| Bumper plus e-stop | Last-resort stop |
| Wheel encoders (and a gyro, optionally) | Distance and heading between markers |
| Optional: depth camera or 2D camera | Load or pallet detection |

**Where AI is actually useful on an AGV**

| Module | Use | Priority |
|---|---|---|
| **Pallet or cart detection** | Camera plus a small detection model to find pockets or cart legs | Useful if you carry pallets |
| **Obstacle or person classification** | Distinguishes a person from a box so the AGV slows or waits | Optional add-on |
| **Fleet scheduling** | Optimization solver for task assignment and traffic | Useful above about 5 vehicles |
| **Predictive maintenance** | Anomaly detection on battery and motor data | Optional, good later |
| **Navigation and SLAM** | Not needed | Skip, because the path is fixed |

**Compute and software**
- A PLC or industrial controller handles motion, guidance, and safety, so no heavy GPU is needed.
- A small embedded module (such as a Jetson Nano or Orin class) can run the vision models if you add them.
- The fleet manager runs on a server, which handles routing, traffic rules, and WMS/MES integration.

**Safety split**
- Certified, non-AI hardware does all the stopping: safety scanners, bumper, e-stop, safety PLC (ISO 3691-4, PL d).
- AI, if used, only helps with decisions such as detecting a pallet or slowing near people.

**Build order**
1. Get path following and docking accurate first, because most AGV problems are guidance problems.
2. Add the safety scanners with proper zones.
3. Add fleet and traffic management.
4. Add vision and maintenance models last.

**AGV vs AMR on AI**
- AGV: AI is optional and small, and a good first design needs almost none.
- AMR: AI is central, because the vehicle perceives and plans on its own.

If your routes are fixed and the layout rarely changes, an AGV is cheaper and easier to certify. Tell me your payload and route layout and I can say which guidance method fits best.

# Concepts
I read this as concept ideas for the vehicle itself, from the AGV and AMR types we covered. Here are six, from simplest to most advanced.

| # | Concept | Idea | Navigation | Payload | Best for |
|---|---|---|---|---|---|
| 1 | **Low-profile cart carrier** | Flat base that drives under a cart or shelf, lifts it, and carries it | AMR (lidar SLAM) or QR markers | 250 to 500 kg | Line-side supply, tote and rack moves |
| 2 | **Modular base + swappable top** | One standard base, with conveyor, lift, hook, or arm modules bolted on top | AMR | 250 to 1000 kg | Changing needs, one fleet doing many jobs |
| 3 | **Hybrid AGV/AMR** | Follows fixed lanes (tags or tape) on main routes and navigates freely in open work zones | Markers plus lidar | 300 to 1000 kg | Mixed layouts where lanes are predictable but ends are not |
| 4 | **Milk-run tugger** | Small tractor pulling a train of carts on a timed loop | AGV or AMR | 1 to 5 tonnes towed | High-volume part delivery to many stations |
| 5 | **Heavy platform AGV** | Wide low-speed platform for engines, battery packs, or car bodies | AGV (laser or wire) | 1.5 tonnes and up | Assembly lines needing millimeter docking |
| 6 | **Mobile manipulator** | AMR base with a robot arm that loads or unloads machines | AMR plus docking | 5 to 10 kg arm, 150 to 600 kg base | Machine tending, kitting |

**Two cross-cutting concepts**
- **Smart-cart idea:** put the intelligence in the base and keep the carts dumb and cheap, so one robot serves many carts.
- **Small fleet idea:** use many small, cheap robots instead of a few large ones. This gives redundancy and flexible capacity, but needs good traffic management.

**What I would pick for a first design**
Concept 2, the modular base with a swappable top. Start with a cart-carrier top module. It matches how the MiR family works, keeps the mechanical design simple, and lets you add new top modules later without redesigning the vehicle.


# For motor control, what driver architecture is needed? (ESC with PWM, H-bridge, MOSFET array)
These three options aren't alternatives at the same level, because they describe different layers. The motor type decides the architecture.

| Option | What it really is | Use on an AMR? |
|---|---|---|
| **PWM** | The control signal that sets average voltage, used by every option below | Always, but it is not an architecture by itself |
| **H-bridge** (4 MOSFETs) | Drives a brushed DC motor in both directions | Only for small brushed motors, such as light AMRs or auxiliary actuators like lifts |
| **3-phase MOSFET bridge** (6 MOSFETs, 3 half-bridges) | Drives a BLDC or PMSM motor | The standard for drive wheels |
| **ESC** (hobby electronic speed controller) | A packaged 3-phase bridge with simple firmware | Fine for prototypes, not for production: no encoder feedback, no safety functions, weak low-speed control |

**What an industrial AMR drive needs**
- **Motor:** BLDC or PMSM hub or geared motor, usually 24 or 48 V.
- **Power stage:** a 3-phase inverter of six MOSFETs (or GaN/IGBT at higher power).
- **Gate driver:** dedicated driver ICs with dead-time, shoot-through protection, and under-voltage lockout.
- **Control:** field-oriented control (FOC) with an encoder or resolver, because it gives smooth, precise low-speed motion and torque control.
- **Sensing:** phase or shunt current sensing, DC bus voltage, and motor temperature.
- **Protection:** overcurrent, over-temperature, and over/under-voltage trips, plus handling for regenerative braking so the bus voltage doesn't spike.
- **Safety:** a Safe Torque Off (STO) input wired to the safety PLC, so the safety layer can cut torque independently of the control software.

**Typical layout for a two-wheel differential AMR**
- One motor controller per drive wheel, each with its own 3-phase bridge, running a speed loop commanded by the main computer over CAN, EtherCAT, or similar.
- A common option is to buy integrated servo drives (with STO and encoder interfaces built in) instead of designing the bridge yourself.

**Recommendation**
- **First prototype:** buy a ready motor controller (FOC with CAN and STO). It saves months, and the safety functions are already certified.
- **Custom design:** use a 3-phase MOSFET array with gate drivers and FOC. The TI 48 V BLDC reference design I linked is a good starting point.

Tell me your motor type and payload, and I can size the MOSFETs, current rating, and braking approach.

