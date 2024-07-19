# Leverege IoT Device Backend

## Business context - What was the company you were at, what was the broader landscape, what was the technical/business problem that needed to be solved?
- Leverege: IoT consulting firm -> internal products
- RecovR: Flagship product
  - Asset tracker with: gps, lte, ble/wifi, battery
  - "Sold" to car dealerships for lot management, could sell through to car buyers as theft recovery solution
  - Innovation: lot management as revenue instead of expense
- Hardware partner: built the devices and wrote the firmware, Leverege managed device backend and user-facing applications
- Existing leverege platform: whitelabel asset tracking UI
- New leverege UI: react-native app (for car buyers to interact w/their theft recovery device)

## What was required to get sign-off from leadership, peers, and your team to move forward with the solution?
- Established trust w/leadship when I was working in our consulting/platform stack
  - A notable task is when I refactored one of our backend services to move from MySQL to Postgres
  - Altered the queries but also helped improve the service code quality and consistency and found a few undetected bugs
- After partnership formed with our hardware partner, I was selected to build the backend with another of my colleages 
  - My focus was primarily on business logic
  - His focus was primarily on dev ops
- Followed demo driven development
  - Aligned with our hardware parters on key integration points
  - Once both parts were ready we would have "one-roofs" where we'd connect the pieces and debug together
  - Ran through a suite of pre-defined tests that we'd share the results from w/management
  - Also made a lot of loom videos to showoff new features

## What were the requirements and what made it a complex technical problem?
- Devices checked in every 15 minutes
  - Report telemetry data
  - Send back commands
- Devices needed to be securely provisioned
  - Process large sets (~50k items) of manufacturing and shipment logs
  - Many tasks to be done for each log type w/task dependencies
- Devices needed over-the-air firmware upgrades
  - Each processor (mcu, ble, lte) had its own firmware
  - Could only upgrade one component at a time
  - Needed to ensure we always had a known set of firmware components on the device at any given time (ensure devices only running well-tested combinations)
  - Needed to minimize the number of upgrade steps since devices were battery powered
- User interfaces needed to interact with the devices
  - Present telemetry data (location, movement alerts, battery level)
  - Set device state (lock movement, enable theft recovery)
- Locationing relied on multiple sources
  - GPS not always available
  - Devices configured as locators could be nearby other devices configured as beacons with known positions
    - Detected nearby via ble scans which could return multiple nearby devices with varying RSSI
  - If neither GPS nor nearby devices available, would fallback to cell tower approximate positioning
- Devices did not have guaranteed communications
  - Cellular or wifi could be unavailable (e.g. in a parking garage)
  - Needed detect widespread comms failures (devices were new hardware + new firmware)
- Needed to manage multiple device types that could run with different configuration profiles
  - The same hardware was initially used as both a locator and a beacon during early development
  - Eventually more focused hardware was developed for beacons (e.g. not battery powered)
- Devices provisioned to different locations had different config files
  - e.g. wifi creds at dealership 1 vs dealership 2

## What was the high-level solution and what made it innovative?
- Built a backend server to manage all device and user interactions
- Started simple, avoided any early abstractions until we achieved our POC.
- Once POC achieved and we had developed a strong understanding of our use cases, we generalized.
- Split the single service into multiple services
  - Generic device server:
    - Core backend logic handled by server internals
    - Device behavior defined in device profiles which define:
      - Which services are enabled (e.g. firmware, MVNO comms, log uploaders)
      - Custom handlers for unique logic (e.g. device reported a heartbeat, device detected as not reporting, device added to a new group)
      - Everything else specific to each hw/fw/comms instance.
  - Generic firmware over-the-air update server:
    - Core upgrade pathing logic
    - Used by generic device server
    - Configurable upgrade strategies (e.g. application based vs component based)
    - Semver dependency definitions at application and/or component level

## What were the details of the solution? Architecture diagrams, technologies used.
- Javascript, Postgres, Sequelize, Express, Google Cloud (GKE, Storage), Redis/Redlock, Prometheus, Grafana
- [Device Profile Overview](./device-profiles.png)
- [Device Profile Notes](./device-profiles.png)

## What were the trade-offs: Why did you choose certain technologies and not others?
- Used established company tech stack
- No clear reason to deviate
- Easier to onboard new devs to project
- In retrospect
  - Typescript would have been valuable
  - Koa has lower latency than Express
  - Use well typed ORM (e.g. MikroORM)

## What were some things you tried that failed?
- Initial log uploaded was not robust enough
  - Expected each task to complete successfully
  - Not a lot of retry / resume / error handling behavior
  - No throttling to external apis
  - Lacked observability
- Created a jobs mechanism
  - Jobs split into batches
  - Jobs tracked in postgres w/status and stage info
  - Introduced better error/retry and throttling for each task
  - Implemented job re-run / resume mechanisms for widespread issues

## Other challenges
There were plenty of early stage challenges that were not foreseen in the beginning but we were able to overcome. We ended up doing a lot more firmware upgrades than we projected as early stage firmware is hard to get right the first time, and many bugs only uncover themselves after the device burns in for a few months. One notable bug, the “time bomb” bug as we called it, caused the devices to brick due to a timer overflowing the max int size and crashing the firmware (also disabling the watchdogs). The root cause of this bug was not immediately obvious as many of our devices would just suddenly go offline and never come back on. To help debug this, I analyzed our fleet of devices to find any pattern between the devices suddenly going offline. The pattern that helped us find the root cause was noticing all devices that were going offline were actively reporting to the backend for ~50 days, then went dark. This made us suspicious of something related to a time-based process failing not-so gracefully. After brainstorming with my team, one of the seasoned engineers put together the significance of 50 days: 2^32 milliseconds = 49.7 days. After sharing our findings with our partner, they confirmed it was a timer overflow issue, scrambled to make a new firmware release, identify which devices needed to receive the new firmware the soonest, and then we rolled it out.

To make matters more complex, this was our first large-scale firmware rollout, and when we pushed the new firmware to one of our first dealerships, many of the devices did not upgrade. We had previously performed a test on ~100 internal devices and all devices upgraded as expected. However, now that we were trying to upgrade ~10K devices, something was breaking down. This was a stressful moment as upgrading the fleet was my responsibility and it was not working – and we only had a few weeks left before all our devices would get bricked. I combed through all my code, firmware configurations, and logs – hunting for obscure edge cases that could explain this, but there was no obvious bug. After retrieving one of the devices that failed the firmware upgrade and chatting with our MVNO, we found another problem. Since so many devices were trying to pull the firmware files at the same time, the cellular band our devices were provisioned to collectively exceeded the bandwidth and the devices experienced firmware file packet-loss. As the devices did not have any retry mechanism for packet-loss, the upgrades failed. For a near term solution, I added a throttling mechanism to limit the number of active firmware upgrades per cell tower while the firmware team improved their packet loss handling. Between these two solutions, we were able to upgrade all the devices and avoided bricking our entire fleet.

Once we got over these initial challenges, the product ran smoothly and rollouts continued to scale. Our teams expanded the product offering and got into a new key tracking product – adding an additional revenue stream for our companies.

## How did you measure success? What did you learn from this?
- Car dealerships happy with lot management solution
- Car dealerships successfully sold devices to car buyers
- Recovered a handful of stolen cars
- Scaled to 250k devices checking in every 15-60 minutes
- Never bricked the fleet

## What were the major lessons you learned - technical or team-wise?
- Demo driven development is effective
- Don't generalize too early
- System monitoring is a good investment