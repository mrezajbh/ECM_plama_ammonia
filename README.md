Purpose and scope
This notebook implements a comprehensive equipment-sizing workflow for the integrated PEM electrolysis plus DBD non-thermal plasma ammonia synthesis concept. It is designed to translate simulation stream results and equipment lists into consistent mechanical and thermal sizing metrics for major plant items (valves, vessels, turbomachinery, pumps, heat exchangers, reactors, electrolyzer, and combustion hardware). The sizing outputs are structured to support subsequent costing, comparison across scenarios, and reproducible updates when stream conditions or design assumptions change.
Inputs
The workflow reads three Excel files:
1.	Final_equipment_completed.xlsx: equipment lists by category (each worksheet corresponds to an equipment class used in the sizing routines).
2.	WBB5F2.xlsx: stream table(s) containing process conditions and flow data used to size thermal and rotating equipment.
3.	Book1.xlsx: plasma-specific sheets, including “DBD Plasma Reactor” and “HB vs DBD Comparison,” used to parameterize the plasma reactor sizing and comparisons.
Core data model
After loading, the code stores the inputs in a single dictionary with four main keys:
•	equipment: a dictionary of pandas DataFrames keyed by worksheet name
•	streams: a dictionary of pandas DataFrames keyed by worksheet name
•	plasma: DataFrames for plasma reactor and comparison sheets
•	compositions: optional composition tables if provided in the stream workbook
Thermodynamic and property layer
A ThermodynamicProperties class provides internal property functions used by sizing calculations, including:
•	Ideal-gas heat capacities using Shomate coefficients (NIST form) for H2, N2, NH3, H2O, O2
•	Vapor pressure correlations for water and ammonia (bar)
•	Mixture properties (Cp and gamma) for H2/N2 blends
•	Liquid water density and Cp, latent heats (water and ammonia), and water viscosity
These functions allow the heat-exchanger sizing routines to compute sensible and latent loads, temperature driving forces, and approximate transport properties consistently across the plant.
Sizing modules and what each produces
The notebook implements sizing functions by equipment category, each returning a DataFrame with calculated design quantities and key intermediate variables:
A. Control valves
•	Computes valve sizing metrics such as Cv (or equivalent capacity factor) based on stream flow and pressure drop assumptions.
B. Vessels
•	Estimates vessel volume and geometric parameters (diameter, length, holdup) based on inventory/retention assumptions, phase, and allowable superficial velocities or residence times where applicable.
C. Turbines
•	Estimates shaft power and related parameters from inlet and outlet conditions and assumed isentropic or polytropic efficiency, supporting power-block and expander sizing.
D. Compressors
•	Estimates compression power from pressure ratios, inlet conditions, and efficiency assumptions (polytropic or isentropic formulation). Used for recycle compression and other compression services.
E. Pumps
•	Computes hydraulic and shaft power based on volumetric flow, head, and efficiency, supporting BOP pump sizing.
F. Heat exchangers and specialized exchanger types
The notebook includes a general heat-exchanger sizing wrapper and internal utilities:
•	LMTD calculation and correction (including a TEMA-style F-factor routine)
•	Shell diameter and shell-and-tube sizing helpers
•	Dedicated sizing subroutines for condenser (process), condenser (utility), WHRU, cryogenic exchanger, gas recuperator, heater, cooler, and a default utility exchanger model
The outputs typically include duty, temperature approaches, required area, and indicative geometry metrics (shell diameter, number of tubes or passes depending on the selected model).
G. Reactors and special units
•	Plasma reactor sizing: uses plasma-sheet inputs and plant duty requirements to size the DBD unit (power-related and throughput-related sizing fields are produced for reporting).
•	Haber-Bosch reactor sizing may be included for benchmarking logic in the comparison sheet, if activated.
•	Combustion chamber sizing: computes sizing metrics based on air and fuel service conditions.
H. Electrolyzer sizing
•	Sizes the PEM electrolyzer based on hydrogen production targets and assumed specific energy consumption and operating factors, returning capacity and power-related fields for integration.
Outputs
The notebook exports:
•	An Excel results workbook (default naming pattern uses an output prefix and writes multiple sheets, one per equipment category).
•	A text report summarizing key sizing results and counts per equipment category.
It also creates an output directory and copies the original Excel inputs into that folder to preserve full reproducibility.
How it is intended to be used
This notebook is the deterministic sizing backbone. When stream conditions, recycle ratio, or plasma duty assumptions change, you only update the input workbooks (or the file paths) and re-run to regenerate a consistent set of equipment sizing outputs.
2.	senfin.ipynb (integrated techno-economic analysis with dynamic equipment costing and sensitivities)
Purpose and scope
This notebook implements an integrated techno-economic model for the plasma-based ammonia plant using dynamic equipment costing correlations rather than fixed, hardcoded costs. The code combines:
1.	Correlation-based purchase-cost estimation for major equipment (sized by capacity, area, power, or volume)
2.	Plant-level capital investment calculation (installed cost structure)
3.	Operating cost and revenue models derived from process material and energy flows
4.	Economic indicators and levelized metrics
5.	Parametric sensitivity studies to identify dominant drivers and operating envelopes
Declared material and product basis
The code is structured around explicit stream-based inputs for material consumption and product formation, including:
•	Water to PEM electrolyzer (stream basis)
•	Nitrogen feed (either purchased or via on-site ASU, depending on scenario)
•	Geothermal steam as the primary energy source (treated as an energy supply, not a consumed commodity)
Products and coproducts include liquid ammonia, oxygen, net electricity export, and district heating.
Cost indexing and adjustment
A CEPCI adjustment utility updates equipment purchase costs between reference years. This allows the model to be re-based to newer years while keeping the underlying correlations consistent. The approach is used throughout the equipment-cost modules so that sensitivity runs remain internally consistent.
Equipment costing layer
The notebook defines a set of correlation-based costing functions for the major unit operations:
•	Control valve costing
•	Vessel costing
•	Turbine costing
•	Compressor costing
•	Pump costing
•	Heat exchanger costing
•	PEM electrolyzer costing
•	DBD plasma reactor costing
•	Haber-Bosch reactor costing (benchmark or comparison logic)
•	Combustion chamber costing
A wrapper function computes all equipment costs for a given scenario, compiles results into structured tables, and extracts key cost contributors.
Capital investment model
The capital module aggregates equipment purchase costs and applies installation and indirect factors to estimate:
•	Fixed capital investment and total capital investment (TCI) according to the model structure used in the paper
The model is designed so that changes in equipment sizing or technology assumptions propagate directly into TCI.
Operating cost model
The OPEX module estimates variable and fixed operating costs from:
•	Feedstock costs (notably nitrogen depending on purchase vs ASU scenario)
•	Utilities and electricity balance effects
•	Labor, maintenance, and other fixed charges using parameterized factors consistent with the manuscript methodology
This structure is intended for scenario comparison and sensitivity rather than a site-specific EPC bid.
Revenue model
The revenue module accounts for:
•	Ammonia sales
•	Oxygen coproduct credit
•	Electricity export (or import penalty)
•	District heating credit where applicable
This is consistent with the polygeneration framing of the study and enables direct evaluation of how integration choices shift the business case.
Economic indicators and levelized metrics
The notebook calculates standard project KPIs and reporting metrics used in the manuscript, including:
•	NPV and payback period
•	IRR (where enabled in the configuration)
•	LCOA and levelized power cost (LPCTotal)
These metrics are recalculated for each scenario and each sensitivity point.
Scenario structure and sensitivities
The main routine runs at least two high-level scenarios:
1.	Nitrogen purchase (no ASU capital)
2.	On-site ASU (includes ASU capital)
In addition, the notebook includes dedicated sensitivity routines that sweep key drivers and regenerate economics and levelized outcomes, such as:
•	Steam or geothermal supply sensitivity (resource strength and power availability effects)
•	PEM-related sensitivities (efficiency and cost assumptions)
•	Purge sensitivity (loop losses and separation/recirculation impacts)
•	Kalina-cycle sensitivity (composition or configuration parameters if enabled)
Plotting functions generate publication-ready figures for cost breakdowns, economic indicators, cashflow structure, material balance, equipment cost breakdown, and comparative sensitivity panels.
Outputs
The notebook writes structured Excel outputs (economic summaries and sensitivity tables) and saves figures at publication resolution. The outputs are organized to support direct transfer into manuscript tables and to enable traceability across scenario runs.
How it is intended to be used
This notebook is the integrated economics and sensitivity engine. When you update technology assumptions, pricing, or a key process parameter (such as recycle ratio or plasma duty), the model recalculates equipment costs, TCI, OPEX, revenues, and levelized metrics consistently across the entire chain, allowing rapid re-basing to new years, price sets, or location assumptions.

