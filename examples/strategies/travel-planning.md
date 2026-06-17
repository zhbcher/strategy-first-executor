# Strategy: Travel Planning

## When to Use

Planning a multi-city trip with flights, trains, hotels, and activities. Optimizing for budget, time, or experience quality. Output is a day-by-day itinerary.

## Typical Policy

```yaml
max_retry: 3
max_replan: 2
phase_timeout: 20m
max_total_runtime: 60m
```

## Strategy Template

```yaml
goal: "Plan a {N}-day trip covering {cities}, optimizing for {budget|time|experience}, with day-by-day itinerary and cost estimate"

constraints:
  - id: all_cities_covered
    type: semantic
    condition: "every city in the itinerary has at least one full day of planned activities"
    verify: manual
  - id: transport_feasible
    type: semantic
    condition: "all inter-city transport connections are realistic (train <5h or flight exists, no impossible same-day transits)"
    verify: manual
  - id: budget_balanced
    type: metric
    condition: "estimated total cost within ±15% of target budget {target_budget}"
    verify: exec: python -c "
import json, sys
d = json.load(open('artifacts/itinerary.json'))
total = d.get('total_estimate', 0)
target = {target_budget}
margin = target * 0.15
if abs(total - target) > margin:
    print(f'Budget: {total} (target: {target} ±{margin:.0f})')
    sys.exit(1)
print(f'Budget: {total} (within range)')
"
  - id: daily_rest
    type: metric
    condition: "each day has ≤{max_hours} hours of scheduled activities"
    verify: exec: python -c "
import json, sys
d = json.load(open('artifacts/itinerary.json'))
for day in d['days']:
    hours = day.get('activity_hours', 0)
    if hours > {max_hours}:
        print(f'{day[\"label\"]}: {hours}h (max {max_hours})')
        sys.exit(1)
print('All days within activity limit')
"

phases:
  - name: route_planning
    actions: "Determine optimal city order using distance and transport availability. For each inter-city leg, list: transport mode (flight/train/bus), approximate duration, estimated cost. Save to route_plan.json."
    consume: []
    output: [route_plan.json]
    constraints: [transport_feasible]

  - name: city_research
    actions: "For each city: search top attractions, recommended neighborhoods for accommodation, local food, weather. Save to city_briefs.json with per-city sections."
    consume: [route_plan.json]
    output: [city_briefs.json]
    constraints:
      - id: cities_researched
        type: metric
        condition: "city_briefs.json has entries for every city in route_plan.json"
        verify: exec: python -c "
import json, sys
route = json.load(open('artifacts/route_plan.json'))
briefs = json.load(open('artifacts/city_briefs.json'))
route_cities = {c['name'] for c in route['cities']}
brief_cities = set(briefs.keys())
missing = route_cities - brief_cities
if missing:
    print(f'Missing: {missing}')
    sys.exit(1)
print('All cities covered')
"

  - name: build_itinerary
    actions: "Compile route_plan.json + city_briefs.json into a day-by-day itinerary. Each day: morning/afternoon/evening activities, meal suggestions, transport if it's a travel day. Include cost estimates per day. Save to itinerary.json."
    consume: [route_plan.json, city_briefs.json]
    output: [itinerary.json]
    constraints: [all_cities_covered, budget_balanced, daily_rest]

  - name: render
    actions: "Convert itinerary.json into a readable markdown document: trip overview, day-by-day plan, cost breakdown, practical tips. Save to itinerary.md."
    consume: [itinerary.json]
    output: [itinerary.md]
    constraints:
      - id: readable_output
        type: regex
        condition: "itinerary.md has Day 1 through Day {N} sections"
        verify: exec: python -c "
import sys
with open('artifacts/itinerary.md') as f:
    text = f.read()
for i in range(1, {N}+1):
    if f'Day {i}' not in text and f'第{i}天' not in text:
        print(f'Missing: Day {i}')
        sys.exit(1)
print('All days present')
"
```

## Notes

- Budget optimization: set `target_budget` low with `margin: 0.10` for tight enforcement
- Experience optimization: swap `budget_balanced` for `attraction_quality` (semantic: "top-rated attractions prioritized")
- Time optimization: add `transit_time` constraint limiting hours spent in transit per day
- Common replan: route infeasible (no direct flights between cities) → replan with different city order or add a hub city
- For international trips, add a `visa_check` phase or constraint for passport/visa requirements
