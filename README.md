# Winter Road Supply Planner

**Seneca Engineering Competition 2026 · Programming**

Read this whole file before you start coding. Everything you need is in this repo.

---

## 1. Background

Many First Nations communities in northern Ontario have no permanent road. Every winter, roads are built across frozen lakes, rivers and muskeg so trucks can bring in a year of fuel, food and building materials.

These winter roads are only open for a few weeks. Trucks can only cross once the ice is thick enough to hold them. Anything that misses the window has to be flown in, at a much higher cost. Warmer winters are making the season shorter and harder to predict.

> **A note on the data.** The town names and road distances are real. The temperatures, ice, cargo, prices, trucks and planes are **simulated for this competition** and do not describe real conditions in these communities.

---

## 2. Your task

Your team has been hired by a northern trucking company to build a **winter road supply planner**.

For each delivery order, your program decides when the truck can safely leave, how much it can carry, and what has to be flown in. It returns two plans: the **cheapest** plan and the **fastest** plan.

### The user enters

| Input | Example | Notes |
|---|---|---|
| Start town | `ThunderBay` | `ThunderBay` or `SiouxLookout` |
| Destination | `SachigoLake` | `Weagamow`, `MuskratDam` or `SachigoLake` |
| Truck type | `semi_b` | A `truck_id` from `data/trucks.csv` |
| Earliest start | `01022027` | First day the truck or a plane can leave (ddmmyyyy) |
| Cargo items | `medicine: 3,000 kg by 10022027` | One or more items. Each has a **name**, a **weight** in kg, and a **deadline** (the day it must arrive by). See section 4.8. |

Example order: from ThunderBay to SachigoLake on a `semi_b`, starting 1 February 2027, with three items:

| Item | Weight (kg) | Deadline |
|---|---|---|
| medicine | 3,000 | 10 Feb 2027 |
| fuel | 20,000 | 15 Mar 2027 |
| lumber | 15,000 | 15 Mar 2027 |

### The program returns

1. **Cheapest plan:** the lowest total cost. If two plans cost the same, pick the one that arrives first.
2. **Fastest plan:** the earliest day that the *last* item arrives. If two plans arrive the same day, pick the cheaper one.

Each plan shows: which items go by truck and which fly, when each item arrives, the truck's departure date, every stop (fuel bought, days waited), every ice crossing (date, ice thickness, max load, truck weight), every flight, and the total cost.

If no plan is possible, or the input is wrong, return an error that explains why (see section 7).

### Your program must also have

- A **GUI** where the user enters an order and sees both plans
- A **map** of the route
- A **chart** of ice thickness over the season
- **History:** save past plans so the user can look them up and compare them
- **Clear error messages** for bad input and impossible orders
- The **command-line test** in section 7

---

## 3. The route

All roads are two-way. There is only one route between any two towns.

| From | To | Distance (km) | Road type | Speed (km/h) |
|---|---|---|---|---|
| ThunderBay | PickleLake | 530 | paved | 80 |
| SiouxLookout | PickleLake | 263 | paved | 80 |
| PickleLake | WindigoLake | 237 | gravel | 60 |
| WindigoLake | Weagamow | 44 | gravel | 60 |
| Weagamow | MuskratDam | 72 | **winter** | 30 |
| MuskratDam | SachigoLake | 67 | **ice** (34 km ice crossing on Sachigo Lake) | 25 |

```
ThunderBay ──┐
             ├── PickleLake ── WindigoLake ── Weagamow ══ MuskratDam ≈≈ SachigoLake
SiouxLookout ┘    (paved)        (gravel)      (gravel)   (winter road)  (ice crossing)
```

Sections with road type **winter** or **ice** are called **ice sections**. The ice rules in section 4 apply to them. Paved and gravel sections are always open.

Weagamow is North Caribou Lake First Nation. Weagamow, MuskratDam and SachigoLake are the communities you deliver to.

---

## 4. The rules

### 4.1 Driving

- One truck per order, making one trip. The items on the truck can't weigh more than the truck's `max_cargo_kg` in total.
- A section takes `distance_km / speed_kmh` hours to drive.
- A truck drives **at most 12 hours per day**.
- A truck can only stop in a **town**. It can't stop in the middle of a section, so a section must be finished on the same day it starts.
- If a section doesn't fit in the hours left today, the truck stays in town overnight and drives it the next day. This is normal and costs nothing extra.
- The truck can leave its start town on any day from the earliest start date.
- The truck's arrival date is the day it finishes the last section.

### 4.2 The ice

An ice section is **open** on a day only if **both** are true:

1. The mean temperature that day is **0 °C or colder**, and
2. The ice is at least **50 cm** thick.

A truck can drive an ice section on a day only if the section is open **and** the truck's total weight is **not more than the max load** for that day (section 5 shows how to calculate it).

**Truck weight on an ice section** = empty weight + truck cargo + weight of the fuel in the tank when it starts that section.

Diesel weighs **0.85 kg per litre**.

### 4.3 Waiting

- A truck can wait in a town for as many days as it needs (for example, until the ice is thick enough).
- A **waiting day** is a day the truck stays in a town only because the next section is closed or the ice can't hold it. Each waiting day costs the truck's `waiting_cost_per_day`.
- Normal overnight stops (section 4.1) are not waiting days.

### 4.4 Fuel

- The truck starts with an **empty tank** and buys fuel at its start town.
- It can buy fuel only at the towns in `data/fuel_stops.csv`, in whole litres, up to the size of its tank.
- `available_l` is the most a town can sell to one truck. `-1` means unlimited.
- Fuel use = `distance_km × fuel_use_l_per_100km / 100`. The tank can never go below 0.
- WindigoLake has no fuel.

### 4.5 Planes

- Planes fly from the **start town** straight to the destination. Every community has an airstrip.
- A plane can leave on any day from the earliest start date. It **arrives the next day**.
- Each flight costs `cost_per_flight`, no matter how full it is. It can carry up to `capacity_kg`.
- Each plane type can make at most `max_flights_per_day` flights per day.

### 4.6 Cost

**Total cost** = truck cost + flight cost

- **Truck cost** (only if the truck carries cargo) = `cost_per_km × route distance` + fuel bought × price + waiting days × `waiting_cost_per_day`
- **Flight cost** = the sum of `cost_per_flight` for every flight

### 4.7 Arrival

- Every item must arrive **on or before its own deadline**.
- A plan's **arrival date** is the day the **last** item arrives.

### 4.8 Items

- An order has **one or more items**. Each item has a name, a weight and a deadline.
- Each item goes **either all on the truck or all by plane**. You can't put half an item on the truck and fly the rest.
- An item that goes by plane **can** be split over several flights, and one flight can carry several items.
- The truck carries all its items together, so they all arrive on the day the truck arrives. That must be on or before the deadline of **every** item on the truck.
- Deciding which items go on the truck is a big part of the problem. For example, an urgent item might fly while the heavy items wait for thicker ice.

---

## 5. Ice 101

You only need two formulas: temperature gives ice thickness, and ice thickness gives the max weight the ice can hold.

### Step 1: Freezing degree days (FDD)

FDD measures how much cold has built up. Start at `FDD = 0` on the first day in `temperatures.csv` and go day by day:

```
FDD_today = max(0, FDD_yesterday − T_today)
```

`T_today` is that day's mean temperature in °C. A cold day (for example −20 °C) **adds** 20. A warm day (for example +3 °C) **removes** 3. FDD never goes below 0.

### Step 2: Ice thickness (Stefan's equation)

```
h = α × √FDD
```

- `h` = ice thickness in **cm**
- `α` = **2.0** (cm per √(°C·day))

### Step 3: Max load (Gold's formula)

```
P = A × h²
```

- `P` = the heaviest total truck weight the ice can hold, in **kg**
- `A` = **5** (kg per cm²)
- `h` = ice thickness in cm, from step 2

**Quick example:** if FDD = 900, then h = 2.0 × √900 = 2.0 × 30 = **60 cm**, and P = 5 × 60² = 5 × 3600 = **18,000 kg**.

### Constants

| Name | Value |
|---|---|
| α (Stefan) | 2.0 cm per √(°C·day) |
| A (Gold) | 5 kg per cm² |
| Minimum ice to open an ice section | 50 cm |
| Max driving per day | 12 hours |
| Diesel weight | 0.85 kg per litre |
| Plane delivery time | Arrives the day after it leaves |

See `examples/worked_example.md` for a full order solved step by step.

---

## 6. Data files

All files are CSV with a header row. Dates are `ddmmyyyy`.

| File | Columns |
|---|---|
| `data/map.csv` | `from`, `to`, `distance_km`, `road_type` (paved, gravel, winter, ice), `speed_kmh` |
| `data/temperatures.csv` | `date`, `mean_temp_c` (one row per day, 1 Nov 2026 to 30 Apr 2027) |
| `data/trucks.csv` | `truck_id`, `name`, `empty_weight_kg`, `max_cargo_kg`, `fuel_tank_l`, `fuel_use_l_per_100km`, `cost_per_km`, `waiting_cost_per_day` |
| `data/planes.csv` | `plane_id`, `name`, `capacity_kg`, `cost_per_flight`, `max_flights_per_day` |
| `data/fuel_stops.csv` | `town`, `price_per_l`, `available_l` (`-1` = unlimited) |
| `data/orders.csv` | `order_id`, `start`, `destination`, `truck_id`, `earliest_start`, `items` (sample orders for your GUI and testing; `items` is written the same way as on the command line, separated by spaces) |

Read the data from these files. Don't hard-code the numbers: the judges may test with changed data.

---

## 7. Command-line test

Besides the GUI, your program must run from the command line in exactly this format:

```
$ winterhaul START DESTINATION TRUCK ddmmyyyy ITEM [ITEM ...]
```

- The date is the **earliest start**.
- Each **ITEM** is written as `name:kg:ddmmyyyy`, where the date is that item's deadline. Names have no spaces or colons.

Example:

```
$ winterhaul ThunderBay SachigoLake semi_b 01022027 medicine:3000:10022027 fuel:20000:15032027 lumber:15000:15032027
```

Your README must explain how to run it (for example `python winterhaul.py ...` or `java -jar winterhaul.jar ...`).

### Output

Print **only** JSON in this format to the screen. This example is part of our answer to test order T2 (`fuel:20000:15032027 lumber:18000:15032027`), shortened:

```json
{
  "Delivery impossible": false,
  "Cheapest plan": {
    "Departure date": "11022027",
    "Arrival date": "12022027",
    "Truck cargo (kg)": 20000,
    "Flown cargo (kg)": 18000,
    "Items": [
      {"Item": "fuel", "Weight (kg)": 20000, "Deadline": "15032027", "Sent by": "truck", "Arrival date": "12022027"},
      {"Item": "lumber", "Weight (kg)": 18000, "Deadline": "15032027", "Sent by": "plane", "Arrival date": "04022027"}
    ],
    "Stops": [
      {"Town": "ThunderBay", "Fuel bought (L)": 400, "Waited (days)": 0},
      {"Town": "PickleLake", "Fuel bought (L)": 56, "Waited (days)": 0}
    ],
    "Ice crossings": [
      {"Section": "Weagamow-MuskratDam", "Date": "12022027", "Ice thickness (cm)": 85.1, "Max load (kg)": 36222, "Truck weight (kg)": 36057}
    ],
    "Flights": [
      {"Plane": "large_plane", "Date": "01022027", "Cargo (kg)": 6000}
    ],
    "Total cost ($)": 70949.2
  },
  "Fastest plan": { "...same fields as Cheapest plan..." }
}
```

- `Items` lists every item in the order. `Sent by` is `"truck"` or `"plane"`.
- `Departure date` is `null` if no truck is used (everything is flown).
- `Stops` lists every town where the truck bought fuel or waited.
- `Ice crossings` lists every ice section the truck drives.
- `Flights` lists every flight, and is `[]` if nothing is flown.
- Round ice thickness to 1 decimal, kg to whole numbers, and cost to 2 decimals.

If no plan is possible, or the input is wrong, print:

```json
{
  "Delivery impossible": true,
  "Reason": "Unknown truck type: rocket"
}
```

### Check your answers

`examples/test_orders.csv` has test orders, and `examples/expected_outputs/` has our answer for each one.

**Your plan does not have to match our numbers exactly.** It has to follow the rules. If you find a valid plan that is cheaper or faster than ours, that's better.

---

## 8. Deliverables

Both are due at **4:00 PM**.

**#1 GitHub repo**, named `[TeamName]_Programming_Code`, submitted as a link and a .zip. Its README must include:
- Team name and members
- Project name
- How to install and run the program (GUI and command line)
- The key parts of your code, to help the judges

Keep your repo private until submissions close.

**#2 Presentation**, named `[TeamName]_Programming_Presentation`, as .pptx **and** .pdf (no videos), **7–10 minutes max**. It should:
- Walk the judges through your solution and design choices
- Include a live demo with test orders
- Explain how your plan helps the communities

Submit both through [submission form] before **4:00 PM**. Late submissions lose **10 points for every 5 minutes**.

---

## 9. Rubric

| Category | Item | Points |
|---|---|---|
| **Strategy & Algorithm** | Solution | 25 |
| | Ease of use / user experience | 10 |
| | Ice window logic (trucks only cross when the ice is ready, within the dates) | 5 |
| | Fallback planning (flying cargo, waiting, refuelling when a truck can't make it) | 5 |
| **Code** | Readability and comments | 10 |
| | Structure (classes and functions) | 5 |
| | Saving and loading past plans | 5 |
| | Command line and JSON output | 15 |
| **Presentation** | Design process and justification | 8 |
| | Clarity, timing and delivery | 3 |
| | Visuals and demo | 3 |
| | Team collaboration and speaking balance | 3 |
| | Answering questions | 3 |
| **Total** | | **100** |

### Penalties

| Penalty | Points |
|---|---|
| Plagiarism | Disqualified |
| Insufficient citation | −50 |
| Late submission | −10 per 5 minutes |
| Absent team member | −25 |

---

## 10. Rules

- Teams of up to 4.
- The design period is **10:00 AM to 4:00 PM**. Nothing can change after 4:00 PM.
- You may use any online resources. Existing code must be **cited** (IEEE) in your presentation, with where and why you used it.
- AI may be used **for research only**, using free, publicly available tools. AI-generated code, slides, text or other deliverables are **not allowed**.
- Plagiarism means disqualification.
- Judges can ask any team member to explain any part of your code.
- Questions during the day: the Discord server, or find Mustafa or Amirali.

Good luck!

---

## References

[1] "Stefan's equation," Wikipedia. [Online]. Available: https://en.wikipedia.org/wiki/Stefan%27s_equation

[2] "Ice road," Wikipedia. [Online]. Available: https://en.wikipedia.org/wiki/Ice_road

[3] Government of Ontario, "Northern Ontario winter roads." [Online]. Available: https://www.ontario.ca/page/northern-ontario-winter-roads

[4] "Ontario Highway 599," Wikipedia. [Online]. Available: https://en.wikipedia.org/wiki/Ontario_Highway_599

[5] Teach For Canada, "Going North on the Ice Road." [Online]. Available: https://teachforcanada.ca/en/stories/going-north-on-the-ice-road/
