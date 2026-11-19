# PowerTrace — PCB Current & Thermal Risk Analyzer

**PowerTrace** is a Python-based KiCad 9 plugin that helps PCB designers identify potential
current-carrying and thermal-risk issues in power nets before manufacturing.

The plugin analyzes the active PCB layout, extracts copper-track and via geometry, estimates
electrical behavior for user-defined currents, highlights risky segments, and generates an
actionable report.

---

## Features

- Extract nets, tracks, vias, pads, and copper-layer information from the active KiCad PCB.
- Assign expected continuous and peak current values to selected power nets.
- Calculate approximate trace resistance, voltage drop, current density, and temperature rise.
- Detect potentially undersized traces and high-risk power-delivery segments.
- Highlight risky nets and copper segments directly in the KiCad PCB Editor.
- Generate a risk report with recommended actions.
- Export analysis results to CSV.
- Support configurable copper thickness, ambient temperature, safety margin, and minimum trace width.

---

## Plugin Idea

PCB designers often route power nets without knowing whether every track and via can safely
carry the required current. While KiCad provides design-rule checks for geometry, it does not
automatically relate expected current, copper geometry, layer thickness, and thermal assumptions
into an easy-to-understand risk report.

PowerTrace fills this gap by providing an early warning system for power-related PCB issues.

For example, if a 5 V rail is expected to carry 2 A continuous current, PowerTrace scans all
tracks and vias on that net. If it detects a narrow 0.25 mm trace on a 1 oz copper layer, it
flags the segment, estimates its resistance and temperature rise, and recommends corrective
action such as increasing trace width or adding a parallel route.

---

## Electronics Concepts Used

- Current-carrying capacity of copper traces
- Trace resistance and voltage drop
- Current density
- Copper thickness and effective conductor cross-section
- Approximate resistive heating and temperature rise
- Via current capacity
- Power-delivery network analysis
- Safety margins for electrical and thermal design

## Computer Science Concepts Used

- Python plugin development for KiCad
- Object-oriented modular architecture
- Graph-based modeling of PCB nets
- Data extraction and transformation
- Rule-engine design
- Configuration handling using JSON
- CSV report generation
- GUI integration with KiCad
- Unit testing and automated analysis

---

## Architecture

```text
KiCad PCB Editor
        |
        v
PowerTrace Action Plugin
        |
        +-----------------------------+
        |         UI Layer            |
        |  Settings, current input,   |
        |  report viewer, highlights  |
        +-----------------------------+
                     |
        +-----------------------------+
        |      Analysis Engine        |
        |  Net graph builder, current |
        |  model, thermal model,      |
        |  rule checker, reporter     |
        +-----------------------------+
                     |
        +-----------------------------+
        |     KiCad API Adapter       |
        |  Board, nets, tracks, vias, |
        |  zones, footprints, canvas  |
        +-----------------------------+
```

### Module Description

| Module | Responsibility |
|---|---|
| `plugin/powertrace_plugin.py` | Registers the KiCad Action Plugin and starts the analysis workflow |
| `plugin/kicad_adapter.py` | Communicates with the KiCad PCB API and extracts board data |
| `plugin/net_graph.py` | Builds a graph of pads, tracks, vias, junctions, and nets |
| `plugin/current_model.py` | Calculates resistance, voltage drop, current density, and temperature rise |
| `plugin/rule_engine.py` | Applies user-defined electrical, thermal, and geometry rules |
| `plugin/reporter.py` | Creates the analysis report and CSV export |
| `plugin/ui/` | Provides the plugin dialog and user settings interface |
| `config/default_rules.json` | Stores default copper, thermal, and safety-rule settings |

---

## Installation

### Requirements

- KiCad 9.0 or later
- Python 3.8 or later
- Git
- A PCB project opened in KiCad PCB Editor

### Install the Plugin

1. Clone this repository:

```bash
git clone [https://github.com/](https://github.com/)<your-username>/powertrace-kicad-plugin.git
cd powertrace-kicad-plugin
```

2. Install Python dependencies:

```bash
pip install -r requirements.txt
```

3. Copy or link the `plugin` folder to the KiCad user plugins directory.

#### Linux

```bash
mkdir -p ~/.local/share/kicad/9.0/scripting/plugins
cp -r plugin ~/.local/share/kicad/9.0/scripting/plugins/powertrace
```

#### Windows

Copy the `plugin` folder to:

```text
%APPDATA%\kicad\9.0\scripting\plugins\powertrace
```

#### macOS

Copy the `plugin` folder to:

```text
~/Library/Application Support/kicad/9.0/scripting/plugins/powertrace
```

4. Restart KiCad.

---

## Running the Plugin

1. Open a PCB layout in **KiCad PCB Editor**.
2. Go to **Tools → External Plugins → PowerTrace**.
3. Select one or more power nets.
4. Enter the expected continuous current and peak current.
5. Set copper thickness, ambient temperature, and safety margin if required.
6. Click **Analyze Board**.
7. Review the generated risk report.
8. Use **Highlight Risks** to visually identify problematic tracks and vias.
9. Use **Export CSV** to save the report.

KiCad Action Plugins are accessible from the PCB Editor menu and are executed through a Python
class derived from `pcbnew.ActionPlugin`. [16][22]

---

## Testing the Plugin

### Test With the Sample Board

1. Open `samples/demo_board.kicad_pcb` in KiCad PCB Editor.
2. Run **Tools → External Plugins → PowerTrace**.
3. Assign 2 A continuous current and 4 A peak current to the `+5V` net.
4. Run the analysis.
5. Verify that narrow or high-resistance segments are reported.
6. Export the report as CSV and inspect the generated file.

### Run Unit Tests

Install test dependencies:

```bash
pip install -r requirements-dev.txt
```

Run tests:

```bash
pytest
```

The unit tests validate the net-graph builder, current/thermal calculations, rule engine, and
CSV report generation independently of the KiCad GUI.

---

## Configuration

Default settings are stored in:

```text
config/default_rules.json
```

Example configuration:

```json
{
  "copper_thickness_oz": 1.0,
  "ambient_temperature_c": 25,
  "maximum_temperature_rise_c": 20,
  "safety_margin_percent": 20,
  "minimum_trace_width_mm": 0.3,
  "maximum_current_density_a_per_mm2": 20
}
```

Users can modify these values to match their manufacturing requirements and design constraints.

---

## Output Report

The plugin generates a report containing:

| Field | Description |
|---|---|
| Net name | Name of the analyzed electrical net |
| Segment/via | Location or identifier of the analyzed object |
| Track width | Width of the copper segment |
| Copper thickness | Copper weight or thickness used for calculation |
| Assigned current | Current entered by the user |
| Estimated resistance | Calculated resistance of the segment |
| Voltage drop | Estimated voltage drop across the segment |
| Current density | Current per unit copper cross-sectional area |
| Estimated temperature rise | Approximate temperature rise due to resistive heating |
| Risk level | Safe, warning, or critical |
| Recommendation | Suggested corrective action |

---

## Limitations

- The initial version provides an approximate thermal model and should not replace full
  electromagnetic, thermal, or manufacturing analysis.
- Copper thickness, layer stack-up, and manufacturing variations must be configured correctly.
- The plugin is intended as a design-assistance tool, not as a certification or safety-approval tool.
- Advanced features such as automatic current inference and IPC-2152-based thermal modeling are
  planned for future versions.

---

## Future Enhancements

- Automatic current estimation from schematic power symbols and load information.
- IPC-2152-based trace temperature-rise modeling.
- Multi-layer current balancing analysis.
- Power-tree validation.
- Integration with eSim and NGspice simulation results.
- PDF and HTML report export.
- Batch analysis of all power nets.
- Plugin settings persistence across KiCad sessions.

---

## Repository Structure

```text
powertrace-kicad-plugin/
├── README.md
├── DESIGN_DOCUMENT.md
├── requirements.txt
├── requirements-dev.txt
├── LICENSE
├── plugin/
│   ├── __init__.py
│   ├── powertrace_plugin.py
│   ├── kicad_adapter.py
│   ├── net_graph.py
│   ├── current_model.py
│   ├── rule_engine.py
│   ├── reporter.py
│   └── ui/
├── config/
│   └── default_rules.json
├── tests/
│   ├── test_net_graph.py
│   ├── test_current_model.py
│   └── test_rule_engine.py
├── samples/
│   └── demo_board.kicad_pcb
└── docs/
    ├── architecture.png
    └── screenshots/
```

---

## Contributing

Contributions are welcome. Please follow these steps:

1. Fork the repository.
2. Create a new branch.
3. Make changes with clear commit messages.
4. Add or update tests.
5. Run `pytest`.
6. Submit a pull request.

---

## License

This project is licensed under the MIT License.

---

## References

- KiCad 9 PCB Editor documentation: https://docs.kicad.org/9.0/en/pcbnew/pcbnew.html
- KiCad Python plugin documentation: https://dev-docs.kicad.org/en/apis-and-binding/pcbnew/
- KiCad `pcbnew.ActionPlugin` API reference: https://docs.kicad.org/doxygen-python-9.0/classpcbnew_1_1ActionPlugin.html
