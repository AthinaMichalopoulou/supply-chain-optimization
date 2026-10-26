[README.md](https://github.com/user-attachments/files/33970372/README.md)
# supply-chain-optimization
# Production, Distribution and Inventory Optimization

An optimization study of production and delivery planning for a steel producer with three factories and eight customer locations over four planning periods. The factories produce long steel bars, cut them into different finished lengths, and deliver customer orders using a limited vehicle fleet.

The interesting part of the problem is that cutting, production, assignment and routing decisions interact. Choosing a factory affects not just available capacity but also how much distance its vehicle must cover.

## Approach

The models are formulated in Pyomo and solved using Gurobi.

**Direct delivery.** The first mixed-integer model chooses cutting patterns, production quantities, customer assignments and delivery routes. Orders must be delivered in their required period, and each customer can receive shipments from only one factory per period. Each factory has one vehicle with a 10-tonne capacity and at most one route per period.

**Customer inventory.** The second model allows rebars delivered early to be held at customer warehouses for future demand. Inventory balance and warehouse-capacity constraints link decisions across periods. This adds flexibility but also makes the optimization harder to solve.

**Warehouse-capacity sensitivity.** The final experiment scales warehouse capacities and compares dispatch activity, distribution cost and computational effort. The transport-cost objective excludes warehouse rental and inventory holding costs, so the comparison is not a complete investment evaluation.

## Results

| Model | Distribution cost | Solver information |
| --- | ---: | --- |
| Direct delivery | €29,047 | Reported optimal |
| Customer inventory | €18,134 | Feasible solution; optimality not proven |

The inventory scenario's reported cost is approximately **37.6% lower**. Both costs are consistent with the delivery routes recorded in the project report. The route distances and dispatch totals are shown in the results section. The inventory solution is a time-limited feasible result, not a proven global optimum.

Earlier warehouse-capacity experiments reported €29,053 with no storage, €19,553 at 7% of reference storage capacity and €18,134 at full capacity. These values are preserved in [`results/historical_results.csv`](results/historical_results.csv) as outputs from the capacity experiment. The slight difference between the no-storage result and the direct-delivery benchmark is unresolved; using a valid big-M reformulation alone would not explain a different proven optimum.

The results suggest that storage flexibility can reduce dispatch and travel costs, but the potential savings have to be weighed against the real cost of providing storage. This model does not price that trade-off.

## Project files

- [Mathematical formulation](docs/model_formulation.md): sets, parameters, decision variables, objective and constraints, matched to the Pyomo model.
- [Results and interpretation](results/README.md): delivery routes, cost breakdown, customer inventory table and capacity experiment.
- [Optimization notebook](notebooks/supply_chain_optimization.ipynb): implementation and optional solve routines.

## Code and reproduction

The analysis is in [`notebooks/supply_chain_optimization.ipynb`](notebooks/supply_chain_optimization.ipynb). It includes the input data, model definitions, solution checking and export utilities.

Install Python dependencies from `requirements.txt` and configure a working **Gurobi license**. Open the notebook and run the cells from top to bottom. All solve flags are `False` by default so that opening or running the notebook does not launch long optimization jobs. Set `RUN_BASELINE`, `RUN_INVENTORY` or `RUN_SENSITIVITY` to `True` in the configuration cell to run the corresponding model. The inventory run defaults to a 5,400-second limit; sensitivity analysis may solve many such models.

The solution checker evaluates variable bounds, integrality and active Pyomo constraints on a newly computed solution; this is a check against the implemented model, not proof that every modeling assumption is correct. A complete new inventory solution can be exported to JSON for later inspection.

## Limitations

Some computationally intensive optimization runs are presented using previously obtained results. Reproducing them may require extended solver time and access to Gurobi.


- The inventory value is a time-limited feasible solution, not a proven minimum.
- The full type-specific shipment and inventory decisions are not included among the saved results.
- Storage costs are excluded, and the capacity experiments do not establish the best economic warehouse size.
- Large mixed-integer routing models can be slow and may return different feasible incumbents under different time limits or solver settings.

## Acknowledgments

Developed collaboratively during my MSc in Business Analytics at Vrije Universiteit Amsterdam.
