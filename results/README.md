# Results and interpretation

## Distribution costs

| Model | Driven distance | Travel cost | Dispatch cost | Total |
|---|---:|---:|---:|---:|
| Direct delivery | 9,209 km | €27,627 | €1,420 | **€29,047** |
| Customer inventory | 5,638 km | €16,914 | €1,220 | **€18,134** |

The reported inventory-enabled solution costs approximately **37.6% less** than the direct-delivery result. The baseline was reported optimal; the inventory result is a feasible incumbent from a time-limited run, not a proven minimum. The sums above are reconstructed from the route sequences and the distance matrix.

## Delivery routes

The routes are listed below for each period and factory. Distances and the resulting costs are calculated from the instance distance matrix; factory-to-customer assignments follow from the route stops.

| Scenario | Period | Factory | Route | Distance (km) |
|---|---:|---|---|---:|
| Direct | 1 | IJmuiden | IJmuiden → Boenen → Schwerte → Iserlohn → Hagen → Dortmund → IJmuiden | 636 |
| Direct | 1 | Segal | Segal → Neuss → Bochum → Gelsenkirchen → Segal | 413 |
| Direct | 2 | IJmuiden | IJmuiden → Boenen → Schwerte → Iserlohn → Gelsenkirchen → IJmuiden | 629 |
| Direct | 2 | Segal | Segal → Neuss → Segal | 280 |
| Direct | 2 | SouthWales | SouthWales → Hagen → Dortmund → Bochum → SouthWales | 1811 |
| Direct | 3 | IJmuiden | IJmuiden → Gelsenkirchen → Iserlohn → Schwerte → Boenen → IJmuiden | 629 |
| Direct | 3 | Segal | Segal → Neuss → Segal | 280 |
| Direct | 3 | SouthWales | SouthWales → Bochum → Dortmund → Hagen → SouthWales | 1811 |
| Direct | 4 | IJmuiden | IJmuiden → Boenen → Schwerte → Iserlohn → Gelsenkirchen → IJmuiden | 629 |
| Direct | 4 | Segal | Segal → Neuss → Segal | 280 |
| Direct | 4 | SouthWales | SouthWales → Bochum → Dortmund → Hagen → SouthWales | 1811 |
| Inventory | 1 | IJmuiden | IJmuiden → Gelsenkirchen → Boenen → IJmuiden | 572 |
| Inventory | 1 | Segal | Segal → Neuss → Segal | 280 |
| Inventory | 1 | SouthWales | SouthWales → Hagen → Iserlohn → Schwerte → Dortmund → Bochum → SouthWales | 1840 |
| Inventory | 2 | IJmuiden | IJmuiden → Dortmund → Schwerte → Iserlohn → Bochum → IJmuiden | 597 |
| Inventory | 2 | Segal | Segal → Hagen → Segal | 412 |
| Inventory | 3 | IJmuiden | IJmuiden → Gelsenkirchen → Dortmund → Hagen → Iserlohn → Schwerte → IJmuiden | 603 |
| Inventory | 3 | Segal | Segal → Neuss → Segal | 280 |
| Inventory | 4 | IJmuiden | IJmuiden → Boenen → Schwerte → Dortmund → Bochum → Gelsenkirchen → IJmuiden | 603 |
| Inventory | 4 | Segal | Segal → Hagen → Iserlohn → Neuss → Segal | 451 |

The machine-readable route data is in [`routes.csv`](routes.csv). Route-level distance calculations do not on their own establish production feasibility, individual shipment quantities, or full inventory balances.

## Inventory at customers

The inventory-enabled solution carries stock between periods. The following table lists the reported nonzero ending inventory totals in tonnes; all are below the corresponding warehouse limits. No inventory was reported at the end of period 4.

| Period | Customer | Ending inventory (t) | Capacity (t) |
|---|---|---:|---:|
| 1 | Bochum | 0.1923 | 10 |
| 1 | Boenen | 4.1705 | 7 |
| 1 | Dortmund | 0.7932 | 12 |
| 1 | Gelsenkirchen | 2.9807 | 10 |
| 1 | Neuss | 6.1777 | 8 |
| 2 | Bochum | 1.3341 | 10 |
| 2 | Boenen | 2.3677 | 7 |
| 2 | Dortmund | 0.9615 | 12 |
| 2 | Gelsenkirchen | 1.2620 | 10 |
| 2 | Hagen | 3.4254 | 12 |
| 2 | Neuss | 1.0577 | 8 |
| 3 | Hagen | 1.3701 | 12 |
| 3 | Iserlohn | 0.3606 | 9 |
| 3 | Neuss | 5.5887 | 8 |

These are total tonnes per customer, not the type-specific quantities needed to reconstruct every inventory-balance constraint. The source values are also provided in [`reported_inventory.csv`](reported_inventory.csv).

## Warehouse-capacity sensitivity

Selected results from the capacity experiment:

| Warehouse capacity (% of reference) | Reported cost |
|---:|---:|
| 0% | €29,053 |
| 7% | €19,553 |
| 100% | €18,134 |

These are selected recorded scenarios; the available historical data do not establish a continuous cost curve or preserve full model outputs for every run. Results from time-limited runs should be interpreted as reported feasible costs where optimality was not established. The €6 difference between the zero-capacity scenario and the direct-delivery optimum is not explained by the change to a valid big-M formulation alone.

The model includes transport and fixed dispatch expenses, but no charge for warehouse space or inventory holding. Therefore, lower transport cost at larger warehouse capacity does not by itself establish the economically best storage investment.
