# Worked example

This walks through test order **T1** step by step. Use it to check your formulas and your logic.

| Start | Destination | Truck | Earliest start |
|---|---|---|---|
| SiouxLookout | MuskratDam | semi_a | 05012027 |

| Item | Weight (kg) | Deadline |
|---|---|---|
| medicine | 2,000 | 10 Jan 2027 |
| food | 6,000 | 31 Jan 2027 |
| lumber | 4,000 | 31 Jan 2027 |

Command line:

```
$ winterhaul SiouxLookout MuskratDam semi_a 05012027 medicine:2000:10012027 food:6000:31012027 lumber:4000:31012027
```

The truck, `semi_a`, has an empty weight of 14,000 kg, a max cargo of 20,000 kg, a 350 L tank, uses 42 L per 100 km, costs $3.80 per km, and costs $500 per waiting day.

---

## Step 1: The route and driving time

| Section | km | Road | Speed | Hours |
|---|---|---|---|---|
| SiouxLookout → PickleLake | 263 | paved | 80 | 3.29 |
| PickleLake → WindigoLake | 237 | gravel | 60 | 3.95 |
| WindigoLake → Weagamow | 44 | gravel | 60 | 0.73 |
| Weagamow → MuskratDam | 72 | **winter** | 30 | 2.40 |
| **Total** | **616** | | | **10.37** |

The total is under 12 hours, so the truck can do the whole trip **in one day**: it arrives the same day it leaves. The only ice section is Weagamow → MuskratDam.

## Step 2: Fuel

Fuel needed for the whole trip = 616 × 42 / 100 = **258.72 L**. That fits in the 350 L tank, so we buy **259 L** at SiouxLookout ($1.70/L) = **$440.30**.

Fuel left when the truck reaches Weagamow (after 544 km):
259 − 544 × 0.42 = 259 − 228.48 = **30.52 L**, which weighs 30.52 × 0.85 = **25.9 kg**.

If the truck is used, it costs 616 km × $3.80 + $440.30 = $2,340.80 + $440.30 = **$2,781.10** (with no waiting days).

## Step 3: How much the ice can hold

Adding up FDD day by day from 1 November (see README section 5):

| Date | Mean temp (°C) | FDD | h (cm) | Max load P (kg) |
|---|---|---|---|---|
| 05012027 | −24.5 | 1020.7 | 63.90 | 20,414 |
| 10012027 | −22.8 | 1139.9 | 67.52 | 22,798 |
| 13012027 | −16.7 | 1193.8 | 69.10 | 23,876 |
| 14012027 | −18.3 | 1212.1 | 69.63 | 24,242 |

For example, on 5 January: h = 2.0 × √1020.7 = 63.9 cm, and P = 5 × 63.9² = 20,414 kg.

The truck weight on the ice is 14,000 (empty) + its items + 25.9 (fuel):

| Items on the truck | Truck weight (kg) | First day the ice can hold it |
|---|---|---|
| All three (12,000 kg) | 26,026 | 19 January |
| food + lumber (10,000 kg) | 24,026 | 14 January |
| medicine + food (8,000 kg) | 22,026 | 9 January |
| medicine + lumber or food alone (6,000 kg) | 20,026 | 5 January |

## Step 4: The cheapest plan

The medicine must arrive by **10 January**, so we have to choose how it travels:

| Option | Truck | Planes | Total cost |
|---|---|---|---|
| A. All on the truck | Can't arrive by 10 Jan (ice too thin until 19 Jan) | | Not possible |
| B. Medicine + food on the truck by 10 Jan | $2,781.10 | lumber 4,000 kg: 3 small planes = $19,500 | $22,281.10 |
| C. Medicine + lumber on the truck by 10 Jan | $2,781.10 | food 6,000 kg: 1 large plane = $22,000 | $24,781.10 |
| **D. Fly the medicine, truck the rest later** | **$2,781.10** (leaves 14 Jan) | **medicine 2,000 kg: 2 small planes = $13,000** | **$15,781.10** |

Option D is the cheapest. Two small planes leave SiouxLookout on 5 January (1,500 kg + 500 kg) and arrive on **6 January**, before the medicine's deadline. The truck waits for the ice and leaves on **14 January** with the food and lumber, arriving the same day. That's before their 31 January deadline.

The truck doesn't wait in a town, because it simply **leaves** on 14 January. So there are no waiting days to pay for.

## Step 5: The fastest plan

The earliest anything can arrive is:
- **By truck:** leave 5 January, arrive 5 January, carrying at most 6,388 kg (20,414 − 14,000 − 25.9).
- **By plane:** leave 5 January, arrive 6 January.

So the last item can't arrive before 6 January. Our fastest plan: the truck takes the **food** (6,000 kg) on 5 January, and **one large plane** takes the medicine and lumber (2,000 + 4,000 = 6,000 kg) on 5 January, arriving 6 January.

Cost = $2,781.10 + $22,000 = **$24,781.10**, with everything there by **6 January**.

## Step 6: The output

```json
{
  "Delivery impossible": false,
  "Cheapest plan": {
    "Departure date": "14012027",
    "Arrival date": "14012027",
    "Truck cargo (kg)": 10000,
    "Flown cargo (kg)": 2000,
    "Items": [
      {"Item": "medicine", "Weight (kg)": 2000, "Deadline": "10012027", "Sent by": "plane", "Arrival date": "06012027"},
      {"Item": "food", "Weight (kg)": 6000, "Deadline": "31012027", "Sent by": "truck", "Arrival date": "14012027"},
      {"Item": "lumber", "Weight (kg)": 4000, "Deadline": "31012027", "Sent by": "truck", "Arrival date": "14012027"}
    ],
    "Stops": [
      {"Town": "SiouxLookout", "Fuel bought (L)": 259, "Waited (days)": 0}
    ],
    "Ice crossings": [
      {"Section": "Weagamow-MuskratDam", "Date": "14012027", "Ice thickness (cm)": 69.6, "Max load (kg)": 24242, "Truck weight (kg)": 24026}
    ],
    "Flights": [
      {"Plane": "small_plane", "Date": "05012027", "Cargo (kg)": 1500},
      {"Plane": "small_plane", "Date": "05012027", "Cargo (kg)": 500}
    ],
    "Total cost ($)": 15781.1
  },
  "Fastest plan": {
    "Departure date": "05012027",
    "Arrival date": "06012027",
    "Truck cargo (kg)": 6000,
    "Flown cargo (kg)": 6000,
    "Items": [
      {"Item": "medicine", "Weight (kg)": 2000, "Deadline": "10012027", "Sent by": "plane", "Arrival date": "06012027"},
      {"Item": "food", "Weight (kg)": 6000, "Deadline": "31012027", "Sent by": "truck", "Arrival date": "05012027"},
      {"Item": "lumber", "Weight (kg)": 4000, "Deadline": "31012027", "Sent by": "plane", "Arrival date": "06012027"}
    ],
    "Stops": [
      {"Town": "SiouxLookout", "Fuel bought (L)": 259, "Waited (days)": 0}
    ],
    "Ice crossings": [
      {"Section": "Weagamow-MuskratDam", "Date": "05012027", "Ice thickness (cm)": 63.9, "Max load (kg)": 20414, "Truck weight (kg)": 20026}
    ],
    "Flights": [
      {"Plane": "large_plane", "Date": "05012027", "Cargo (kg)": 6000}
    ],
    "Total cost ($)": 24781.1
  }
}
```

Other valid answers exist. For example, the fastest plan could put the lumber on the truck instead of the food, and fly the medicine and food together on the large plane. It costs the same and arrives the same day. Both are correct.
