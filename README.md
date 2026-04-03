**GamePad. End-to-End PCB Design in KiCad**

Custom Symbols · Footprint Engineering · DRC/ERC · IPC-Compliant Output · Gerber Package

A full PCB design project executed entirely in KiCad, from blank schematic through to manufacturer-ready Gerber files. Every decision was made with manufacturing constraints and industry standards in mind.

**1. Schematic design & library familiarization**
Before opening KiCad, I reviewed the full circuit on paper, mapping every signal path, power rail, and component dependency. This offline pass was intentional: it forces architectural thinking before you get distracted by the software. The schematic was then rebuilt from scratch in KiCad's Schematic Editor using standard component symbols from the pre-installed libraries and power symbols for VCC, GND, and regulated supply nets.
Every component was assigned a reference designator, value, and where applicable, a part number. Net labels replaced physical wire routing between distant schematic blocks, keeping the design readable and reconfigurable, a practice consistent with schematic documentation quality expectations outlined in IPC-2612 (schematic documentation requirements for PCBs).
Net integrity was verified at each stage, every pin was either driven, pulled, or deliberately left floating with intent documented.

**2. ERC. Catching errors at the source**
The Electrical Rules Check was run incrementally throughout schematic construction, not just at the end. Every time a new block of components was wired in, ERC was triggered immediately to surface conflicts: unconnected pins, conflicting drive types, missing power flags, duplicate reference designators. Addressing these at the component level is significantly faster than untangling a full-schematic error list at completion.
Common flags addressed included missing PWR_FLAG symbols on power nets, pin type mismatches on passive components, and unconnected input pins on logic devices. Each was resolved with an understanding of why the rule exists, not suppressed. This incremental verification approach mirrors the design review philosophy embedded in IPC-2221B (Generic Standard on Printed Board Design), which emphasizes design validation at each stage of the process rather than a single end-of-cycle check.

**3. Custom symbol creation. The OLED module**
The OLED display module required for this design was not available in the KiCad symbol libraries. Rather than approximating with a generic connector symbol, a shortcut that might cause footprint mismatches downstream, I built a custom symbol using the KiCad Symbol Editor.
Every pin was defined with the correct electrical type: power, ground, bidirectional (SDA), input (SCL), and reset. Pin electrical types directly affect ERC validation, an incorrectly typed pin can mask real drive conflicts or generate false errors that obscure genuine issues. The symbol was drawn to reflect the module's physical block structure, making the interface immediately readable to anyone reviewing the schematic.
The symbol was saved to a project-local library to ensure portability and prevent dependency on global library versions, a practice aligned with IPC-2612 documentation control guidelines.

**4. Footprint assignment, cross-referencing datasheets and distributors**
For each unique component, I cross-referenced three sources: the KiCad footprint library, the manufacturer datasheet (specifically the mechanical drawing and recommended land pattern), and distributor listings on Mouser to confirm stock and lead time. Land pattern dimensions were validated against IPC-7351C (Land Pattern Standard for SMD Components), which defines pad size, pitch tolerance, and courtyard clearance requirements for each component package type.
Footprint selection criteria applied throughout:
•	Package pitch confirmed against manufacturer soldering capability, fine-pitch components below 0.5mm were flagged for assembly risk assessment
•	Courtyard boundaries set to IPC-7351C minimums to prevent component overlap and ensure rework access
•	Through-hole vs. SMD selection weighed against assembly method, rework accessibility, and wave vs. reflow process compatibility
•	Component availability and lifecycle status checked at point of footprint assignment and not after layout
Where two parts were electrically equivalent and mechanically compatible, the lower-cost, higher-availability option was selected. A part that goes out of stock kills a production run and availability is a DFM consideration as much as geometry is.

**5. Custom footprint creation. The two non-standard parts**
Two non-standard components had no adequate footprint match in the KiCad library. Both were built from scratch using the Footprint Editor, working directly from the mechanical drawing in each component's datasheet. Land patterns were constructed in accordance with IPC-7351C guidelines, pad geometries, courtyard expansion, and fabrication layer outlines all derived from the standard's requirements for the applicable package type.
•	Copper pads: Pad size and position from the IPC-7351C recommended land pattern; SMD pads assigned to F.Cu, through-hole pads with drill diameter set to pin diameter plus IPC-recommended annular ring allowance
•	Drill holes: Diameter set per IPC-2221B minimum annular ring requirements to ensure structural integrity post-fabrication
•	Silkscreen (F.Silkscreen): Component outline and pin marker placed per IPC-7351C, offset from pad edges to remain visible post-solder
•	Courtyard (F.Courtyard): Clearance boundary set to IPC-7351C Level B minimums
•	Fabrication layer (F.Fab): True component body outline for assembly documentation and pick-and-place reference
Both footprints were verified by overlaying the 3D model and checking pad positions against the datasheet mechanical drawing before use in the layout.

**6. Manufacturer selection and constraint mapping**
The manufacturer was selected before layout began. The manufacturer's published design rules were loaded into KiCad's Board Setup and cross-checked against the minimum electrical spacing and conductor width requirements defined in IPC-2221B. Where the manufacturer's tolerances were tighter than IPC minimums, the manufacturer's values were used, the tighter constraint always governs.
•	Minimum trace width, set per manufacturer capability, validated against IPC-2221B Table 6-1 current-carrying capacity requirements for the copper weight selected
•	Minimum clearance confirmed against IPC-2221B electrical spacing tables for the board's operating voltage class
•	Minimum drill size & annular ring set to manufacturer capability, checked against IPC-2221B minimum annular ring requirements for the applicable hole class
•	Board edge clearance, copper keepout applied at the manufacturer's specified margin from Edge.Cuts
•	Copper weight selected against current load requirements using IPC-2221B trace width/current tables
All values were entered as hard constraints in KiCad so DRC would automatically flag violations during layout, no relying on manual checks after the fact.

**7. PCB layout and routing**
Component placement was tackled first, grouping by functional block (power, MCU, display, input) and positioning decoupling capacitors as close to IC power pins as physically possible. Placement decisions were made with assembly in mind: components on the same side, orientation standardised to simplify pick-and-place programming, and fine-pitch parts kept away from board edges to reduce handling risk during manufacturing practices consistent with IPC-A-610 acceptability criteria for component placement quality.
•	Power traces routed wider than signal traces, sized to current requirements using IPC-2221B trace width tables
•	Vias sized to manufacturer minimums with annular rings meeting IPC-2221B Class B requirements
•	45° routing angles used throughout, 90° corners create acid traps during etching and are explicitly flagged in IPC-2221B
•	Ground pours added on back copper to provide low-impedance return paths and reduce EMI coupling
•	I²C signal traces kept short and separated from switching power nets to reduce crosstalk
Ground pour was applied last after all routing was complete, with thermal reliefs on through-hole pads to ensure reliable solder joint formation, a direct J-STD-001 (Requirements for Soldering Electrical and Electronic Assemblies) consideration, which specifies acceptable heat transfer conditions for through-hole soldering.

**8. DRC. Validating against the standard before files leave**
The full Design Rules Check was run against the manufacturer constraints loaded in Board Setup, every copper feature on every layer validated simultaneously for clearance violations, unconnected nets, acid trap geometries, silkscreen-to-pad overlap, missing courtyard definitions, and drill-to-copper clearances.
No DRC flag was suppressed without a deliberate, documented reason. This is the point where DFM intent gets validated at the physical level and it maps directly to the pre-fabrication review process described in IPC-2221B. Silkscreen was checked to ensure no markings overlapped exposed copper pads, which would compromise solder mask inspection per IPC-A-610 visual acceptability criteria.
DRC was run multiple times as final refinements were made. A clean DRC result against real manufacturer constraints, not default KiCad tolerances was the gate condition for moving to Gerber generation.

**9. BOM generation and Gerber output**
The Bill of Materials was exported with full component attributes, reference designator, value, footprint, manufacturer part number, and quantity structured to be directly usable by a procurement or assembly team without requiring follow-up clarification.
The surface finish (HASL vs. ENIG) was selected with solderability in mind, ENIG provides a flatter, more consistent surface for fine-pitch components, which J-STD-001 identifies as a factor in achieving acceptable solder joint formation.
The full Gerber package generated:
•	F.Cu / B.Cu - front and back copper layers
•	F.Mask / B.Mask - solder mask layers defining exposed pad areas
•	F.Silkscreen / B.Silkscreen - component labels and board markings
•	Edge.Cuts - board outline for routing and panelisation
•	Drill file (.drl / Excellon format) - all via and through-hole positions and diameters
All files were verified in a Gerber viewer prior to upload, checking layer alignment, aperture definitions, and drill file integrity. The package was then submitted to the manufacturer portal where board specifications were finalised: material (FR-4), copper weight, surface finish, solder mask colour, silkscreen, and controlled impedance requirements where applicable.

**Standards Referenced:**

IPC-2221B - Generic PCB Design Standard (clearances, drill, annular ring, trace sizing)

IPC-A-610 - Acceptability of Electronic Assemblies (placement, solder joint, silkscreen criteria)

J-STD-001 - Soldering Requirements (thermal relief, paste aperture, surface finish)

IPC-2612 - Schematic Documentation Requirements

**Tools Used**

KiCad 7 · Symbol Editor · Footprint Editor · ERC / DRC · Gerber / Excellon output · Mouser / DigiKey component validation · Gerber viewer

