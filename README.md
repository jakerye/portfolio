# Portfolio

Professional and side project summaries.

**Table of Contents**
- [Professional Projects](#professional-projects)
  - [Brightflow AI](#brightflow-ai)
    - [New Platform Architecture](#new-platform-architecture)
    - [Banking Data Sync](#banking-data-sync)
    - [Online Sales Data Parity Investigation](#online-sales-data-parity-investigation)
    - [UI Components Library](#ui-components-library)
  - [Leverege](#leverege)
    - [IoT Device Backend](#iot-device-backend)
    - [Firmware Over-The-Air Upgrades Backend](#firmware-over-the-air-upgrades-backend)
  - [MIT Media Lab](#mit-media-lab)
    - [Plant Research Device Control Software](#plant-research-device-control-software)
    - [Light Controller Software](#light-controller-software)
    - [Light Controller Printed Circuit Board](#light-controller-printed-circuit-board)
- [Side Projects: Software](#side-projects-software)
  - [UnicornTales](#unicorntales)
  - [TandyGram](#tandygram)
- [Side Projects: Construction](#side-projects-construction)
  - [Backyard Garden](#backyard-garden)
  - [Basement Bathroom](#basement-bathroom)
  - [Kitchen Bench](#kitchen-bench)
  - [Porch Repair](#porch-repair)
  - [Master Closet](#master-closet)
  - [Kitchen Pantry](#kitchen-pantry)
  - [Underdeck Roof](#underdeck-roof)

## Professional Projects

### Brightflow AI
#### New Platform Architecture
- Collaborated on new platform architecture to overcome data accuracy, modeling, and access limitations present in legacy architecture.
- Used an "onion" architecture and followed domain driven design (DDD) principles.
- Codebase built as a mono-repo to share domain, framework, and infrastructure across multiple deployed services (e.g. RestAPI, sync workers).
- Codebase well-typed and tested with encapsulated interfaces (e.g. repository implementations hidden from application logic to allow for evolution and various storage strategies).
- Stack: Typescript, Koa, Postgres, MikroORM, AWS (Aurora, S3, MSK, EKS)

#### Banking Data Sync
- Develped sync mechanism to get banking data from Plaid and data enhancements (labelling) from Heron Data.
- Synced bank institutions, accounts, account balances, and transactions.
- Split the fetching and storage of "raw" data from data sources into their own entities from the normalization of the data (decouple extraction from transformation).
- Encountered sync coordination challenges as heron needed to run their own plaid sync before we could get the labelled data.
- Relied on polling based system as webhooks were not reliable enough (external API did not have method guaranteeding webhook delivery).
- Created data checks to run periodically that would pull a sample of data from external sources and validate our internal representation of the data aligned perfectly. 
- Piloted logic in legacy architecture, migrated to new architecture once validated and new platform infrastucture sufficiently developed.

#### Online Sales Data Parity Investigation
- The online sales data in our internal dashboard did not match perfectly with the data in external dashboards (e.g. Shopify, Amazon).
- Isolated the logic from our sync controllers into a test script and validated the discrpancy. 
- Built new scripting logic that only relied on external API responses to reconstruct the shopify/amazon dashboards.
- Collaborated with product team to explicate key metric definitions.
  - Gross Sales = Total Price
  - Net Sales = Total Price - Discounts - Refunds - Shipping - Taxes - Duties - Fees - Tips
- The logic to reconstruct the dashboards was not well documented in offical docs or forums so had to figure out the mappings myself and validate against real data.
  - This was challenging as there were mutiple nested fields with related variables and statuses that caused this to be a more complex mapping than just a 1:1 field extraction.
- Identified key differences in dashboard data sourcing -- Shopify sales dashboard includes all new sales and new refunds that occured during the given period. Our logic only included new sales and excluded any refund handling.
- Refund handling needed to be handled carefully as there are multiple ways to view the refund data.
  - Refunds could be applied to the order entity to get a total net sales figure for a given order, however this loses refund timing information required to reconstruct the sales dashboard in shopify (as shopify includes new orders and new refunds as seperate sales items -- if our internal model did not capture the refund timing information then we would have no way to reconstruct their dashboard).
- Identified innacurate mappings from our internal logic that excluded any refunded or partially refunded shopify orders that prevented those orders from being included in our stored data.
- Identified flaws in the Amazon sync logic that led to our data being stale by excluded updates on orders that transitioned to a 'complete' state -- this flaw is that refunds can occur even after an order is complete and alter the net sales values.
- Shared findings with team and made recommendations on short term fixes (e.g. changing mapping logic is simple) and longer term fixes (e.g. create dedicated entites for refunds with foreign keys to order entities).

#### UI Components Library
- Our frontend app used bespoke components for every piece of our UI. This made presentational consistency challenging and development ineffient. 
- Introduced a basic set of components based on our existing ones and isolated their scope.
- Hosted the components library in a tangential app built with Ladle (lightweight Storybook) for isolated development, design collaboration, and QA.
- Resulted in increased developer efficiency and presentational consistency.

### Leverege

#### IoT Device Backend
- Managed large fleet (>250K) of connected devices to control device state and interface for application logic.
- Devices would check in periorically (15-60 mins) and the server would process data sent by the devices (e.g. location, nearby bluetooth devices, battery level) and send various commands to them (e.g. set operation mode, upgrade firmware) based on application or operational intents.
- Stack: Javascript, Express, Postgres, Sequalize, GCP

#### Firmware Over-The-Air Upgrades Backend
- Generic firmware management server that would store device state and manage firmware releases to a fleet of devices to upgrade their processor firmware through known paths.
- Allowed for the creation of new firmware components (e.g. main controller, bluetooth/wifi controller, cellular controller) with semantic versioning (semver).
- Allowed for the creation of new firmware applications as a composition of firmware component semantic version schemas.
- Each component could specify dependencies on other components, applications could specify dependencies on other applications -- the logic for the upgrade path generation would use either one depending on the upgrade mode configured for the fleet.
- Stack: Javascript, Express, Postgres, Sequalize, GCP

### MIT Media Lab

#### Plant Research Device Control Software
- Ran "climate recipes" that would synthesize environmental conditions over time by setting actuator states (e.g. light channel power modulation, pump on/off, fan on/off) based on sensor feedback (e.g temperature, humidity, water level).
- Stack: Python, Raspberry Pi
#### Light Controller Software
- Synthesized desired light fields by varying intensity of multi-channel light panels
- Used a high-end specrtometer to map each light channel's spectral fields at varying intensities and distances from the light source across sampling planes.
- The spectometer measured intensities across a set of spectral bands.
- Used linear algebra to solve for each channel's power output based on desired spectral field and intensity of a given plane.
- Validated output with spectrometer, achieved 90% accuracy.
- Stack: Python
#### Light Controller Printed Circuit Board
- Printed circuit board (PCB) to control multi-channel light panels
- Designed using modular electical component blocks that had well defined interfaces and footprints for re-use in other circuit boards.
- Stack: Eagle, RaspberryPi, dI2c

## Side Projects: Software
### UnicornTales
- AI powered children's book generator to turn story concepts into illustrated, hard-cover books.
- Stack & Integrations: Typescript, Koa, GCP Firebase (Hosting, Storage, Firestore, Functions, Auth), React, Ant Design, ChatGPT, RPI Print
### TandyGram
- Send postcards by texting photos.
- Stack & Integrations: Python, Django, Postgres, Heroku, React, Bootstrap, Twilio, Lob, Stripe.

## Side Projects: Construction

### Backyard Garden
- Designed and built a large garden (1000 ft2) with patio, retaining wall, raised veggie beds, irrigation system, and power for overhead string lights.
- We built everything ourselves except for tasks that required heavy machinery (i.e. excavation, brick delivery).
- Modelled our yard in 2D layout and created 3D models for raised garden beds
- Stack: Onshape, GardenPuzzle, Google Earth, Lucidchart

### Basement Bathroom
- Built out a rough-in bathroom to have a closed steam shower with tiling, vanity, and other bathroom fixtures.
- Modified plumbing to accomodate vanity location.
- Stack: Photopea

### Kitchen Bench
- Designed and constructed a built-in kitchen bench with power outlets that matched the existing Wainscoting on the surrounding walls.
- Stack: Onshape

### Porch Repair
- Repaired an old porch with rotting floor panels.
- Custom milled bespoke floor tongue in groove (TIG) boards with a router table as old house used non-standard board sizes.
- Sanded and repainted the entire floor.
- Encapsulated old railings (lead paint) with properly rated paint.

### Master Closet
- Added an additional closet to the master bedroom.
- Framed, drywalled, and painted closet. 
- Added bifold doors.

### Kitchen Pantry
- Converted an unused "butlers" entrance to the house into a pantry.
- Removed exterior door with solid wall, added overhead light fixture, drywall and floating shelves.

### Underdeck Roof
- Added a metal roof to the underside of a deck to create a dry outdoor area.
- Roof sloped away from the house and fed into gutters.

