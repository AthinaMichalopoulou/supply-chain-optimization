# Mathematical formulation

The two models combine cutting-stock decisions with factory-to-customer vehicle routing over four periods. The second model adds inventory held at customers. Both are mixed-integer linear programs (MILPs), implemented in `notebooks/supply_chain_optimization.ipynb` with Pyomo.

## Sets and data

| Symbol | Meaning |
|---|---|
| $F$ | Factories: IJmuiden, Segal, South Wales |
| $C$ | Eight customer locations |
| $T=\{1,2,3,4\}$ | Planning periods |
| $R=\{A,B,C\}$ | Finished rebar types, lengths 2.4, 3.6, 4.2 metres |
| $P$ | Cutting patterns generated for 9 m and 12 m long bars |
| $A_f$ | Directed arcs $(i,j)$ over $\{f\}\cup C$, with $i\ne j$, for depot $f$ |

Parameters use the names from the Python notebook:

| Math | Code | Meaning |
|---|---|---|
| $D_{ctr}$ | `D[(c,t,r)]` | Demand (number of rebars) |
| $a_{rp}$ | `Arp[(r,p)]` | Number of type-$r$ rebars in cutting pattern $p$ |
| $w_r$ | `Wr[r]` | Weight of finished rebar $r$, tonnes |
| $W_p$ | `WLP[p]` | Weight of long bar used by pattern $p$, tonnes |
| $K_f$ | `Cap[f]` | Factory production capacity, tonnes per period |
| $d_{ij}$ | `Dist[(i,j)]` | Distance between locations, km |
| $h_f$ | `FSC[f]` | Fixed factory dispatch cost, euros |
| $v$ | `VC` | €3 per driven kilometre |
| $Q$ | `Vcap` | 10-tonne vehicle capacity |
| $H_c$ | `BaseInvCap[c]` | Warehouse capacity, tonnes |

Weights are calculated from rebar diameter 0.057 m and steel density 7.85 tonnes/m³: $w_r=\pi(0.057/2)^2\ell_r\cdot 7.85$. Cutting patterns include all nonempty integer combinations of the three rebar lengths that fit into either stock-bar length.

## Decision variables

| Math | Code | Domain / interpretation |
|---|---|---|
| $x_{fpt}$ | `m.X[f,p,t]` | Nonnegative integer; number of long bars cut with pattern $p$ |
| $y_{fctr}$ | `m.Y[f,c,t,r]` | Nonnegative integer; rebars shipped from $f$ to $c$ |
| $s_{fct}$ | `m.S[f,c,t]` | Binary; customer $c$ served by factory $f$ |
| $u_{ft}$ | `m.U[f,t]` | Binary; factory $f$ dispatches a route |
| $z_{fijt}$ | `m.Z[(f,i,j),t]` | Binary; arc $i\to j$ on factory $f$ route |
| $q_{ftc}$ | `m.Pos[f,t,c]` | Continuous in $[0,|C|]$; visit-order variable |
| $I_{ctr}$ | `m.I[c,t,r]` | Nonnegative integer; ending inventory, inventory model only |

The objective in both models minimizes fixed dispatch and travel costs:

$$\min\ \sum_{t\in T}\sum_{f\in F} h_f u_{ft}+v\sum_{t\in T}\sum_{f\in F}\sum_{(i,j)\in A_f}d_{ij}z_{fijt}.$$

## Direct-delivery model

For every $c,t,r$, demand is met by deliveries in the same period:

$$\sum_{f\in F}y_{fctr}=D_{ctr}.$$

Production matches shipments by rebar type and factory/period, and long-bar production cannot exceed factory capacity:

$$\sum_{p\in P}a_{rp}x_{fpt}=\sum_{c\in C}y_{fctr}\quad(\forall f,t,r),$$
$$\sum_{p\in P}W_p x_{fpt}\le K_f\quad(\forall f,t).$$

Each factory has one 10-tonne vehicle available each period. Shipment quantities are linked to vehicle use and customer assignment:

$$\sum_{c\in C}\sum_{r\in R}w_r y_{fctr}\le Q u_{ft}\quad(\forall f,t),$$
$$y_{fctr}\le D_{ctr}s_{fct}\quad(\forall f,c,t,r),$$
$$\sum_{f\in F}s_{fct}\le 1\quad(\forall c,t),\qquad s_{fct}\le u_{ft}\quad(\forall f,c,t).$$

## Routes (shared by both models)

Each active factory has one departing and one returning depot arc:

$$\sum_{j\in C}z_{ffjt}=u_{ft},\qquad\sum_{i\in C}z_{fift}=u_{ft}\quad(\forall f,t).$$

For every customer, entering and exiting arcs correspond to the factory assignment:

$$\sum_{i\in(\{f\}\cup C)\setminus\{c\}}z_{fict}=s_{fct},\qquad
\sum_{j\in(\{f\}\cup C)\setminus\{c\}}z_{fcjt}=s_{fct}\quad(\forall f,t,c).$$

Miller–Tucker–Zemlin constraints exclude disconnected cycles among customers ($n=|C|$):

$$q_{fti}+1\le q_{ftj}+n(1-z_{fijt})\quad(\forall f,t,\ i,j\in C,\ i\ne j),$$
$$s_{fct}\le q_{ftc}\le n s_{fct}\quad(\forall f,t,c).$$

Routes and shipment links are jointly enforced, so a positive shipment requires a customer visit. Variable domains are as specified above.

## Customer-inventory model

This model **replaces**, rather than retains, the same-period demand equation. It imposes initial inventory and an inventory balance for every customer, period and rebar type:

$$I_{c0r}=0,\qquad I_{ctr}=I_{c,t-1,r}+\sum_{f\in F}y_{fctr}-D_{ctr}.$$

Inventory is nonnegative and limited by each warehouse's total tonnage:

$$\sum_{r\in R}w_r I_{ctr}\le H_c\quad(\forall c,t).$$

The shipment/assignment link is widened to accommodate deliveries for current and future consumption:

$$y_{fctr}\le M_{ctr}s_{fct},\quad
M_{ctr}=D_{ctr}+\left\lfloor H_c/w_r\right\rfloor.$$

The objective, production balance, production capacity, vehicle capacity, customer assignment and routing constraints remain as in the direct-delivery model. The code does **not** explicitly impose zero inventory in the last period; with no post-horizon demand or holding value, zero ending inventory is observed in the reported solution.

## Warehouse-capacity experiment

For each factor $\alpha\in[0,1]$, the experiment substitutes $H_c(\alpha)=\alpha H_c$ into the warehouse-capacity constraint and the shipment big-$M$ expression. All other model components stay the same. The objective excludes rental or inventory holding costs.

The Python functions `solve_baseline`, `solve_with_inventory` and `run_model_with_capacity_factor` implement these equations. For computationally limited MILP runs, a returned feasible incumbent is not necessarily a proven optimum.
