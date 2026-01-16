# secsgem simulator

Simulator tool for SECS/GEM (Semiconductor Equipment Communications Standard / Generic Equipment Model) communication protocol testing.

## Overview

This is a professional SECS/GEM communication protocol simulator designed for testing and validating semiconductor equipment communication layers. The simulator provides a complete implementation of the SECS/GEM protocol with support for SML (SECS Message Language) parsing, CLI tools, and remote management capabilities.

## Features

- **Complete SECS/GEM Protocol Support**: Full implementation of SECS/GEM communication protocol
- **Multiple Data Types**: Support for all SECS data types (Boolean, Number, String, Array, Binary, JSON, List)
- **SML Parser**: Built-in parser for SECS Message Language (SML)
- **Command Line Interface**: User-friendly CLI tool (`sgsim`) for easy operation
- **Remote Management**: Command server for remote control and management
- **Docker Support**: Containerized deployment with Alpine Linux
- **Lua Scripting**: Script execution support using Lua via Lupa
- **Cross-Platform**: Compatible with Python 3.7 through 3.10

## Installation

### Using uv (Recommended)

```bash
# Install uv if you haven't already
curl -LsSf https://astral.sh/uv/install.sh | sh

# Clone the repository
git clone <repository-url>
cd secsgem_simulator

# Install dependencies
uv sync
```

### Using Docker

```bash
# Build the Docker image
docker build -t secsgem-simulator .

# Run the container
docker run -p 4000:4000 -p 5000:5000 secsgem-simulator
```

## Usage

### Command Line Interface

The simulator provides a CLI tool called `sgsim` with the following commands:

#### Start Server

Start the SECS/GEM server daemon:

```bash
sgsim start_server --host localhost --port 4000
```

#### Run Server

Run the server in the foreground:

```bash
sgsim run_server --host 0.0.0.0 --port 4000
```

#### Ping Server

Check if the server is responsive:

```bash
sgsim ping_server --host localhost --port 4000
```

#### Stop Server

Stop a running server:

```bash
sgsim stop_server --host localhost --port 4000
```

#### Run Script

Execute a Lua script:

```bash
sgsim run_script script_name.lua
```

### Sending Test Data

The simulator provides multiple ways to send test data through SECS/GEM protocol:

#### 1. Using Lua Scripts

Create a Lua script with SECS commands:

```lua
secsgem = base.setup("0.1.9")
connection = secsgem.hsms("192.168.1.100", 5000, true, 1)
connection.connect()
connection.send_function("S1F1 W")
connection.disconnect()
```

Run the script:

```bash
uv run sgsim run_script test_send_data.lua
```

#### 2. Common SECS Messages

- **S1F1** (Are You Online): `connection.send_function("S1F1 W")`
- **S2F41** (Host Command): `connection.send_function("S2F41 W < L [2] < A \"START\" > < L > .")`
- **S5F1** (Alarms): `connection.send_function("S5F1 < L [2] < B [1] 0 > < U4 [1] 100 > .")`
- **S6F11** (Event Data): `connection.send_function("S6F11 < L [2] < U4 [1] 1001 > < L [1] < U4 [1] 100 > > .")`

For detailed examples, see `docs/test_data_sending_guide.md`

### Docker Usage

```bash
# Expose ports for communication
docker run -d \
  --name secsgem-sim \
  -p 4000:4000 \
  -p 5000:5000 \
  secsgem-simulator
```

## Project Structure

```
secsgem_simulator/
├── secsgem_simulator/
│   ├── __init__.py              # Package initialization
│   ├── __main__.py              # Module entry point
│   ├── cli.py                   # Command line parser
│   ├── cli_tasks.py             # CLI task implementations
│   ├── command_server.py        # Remote command server
│   ├── server_tasks.py          # Server task processing
│   ├── secs_data.py             # Base SECS data structures
│   ├── secs_data_boolean.py     # Boolean data type
│   ├── secs_data_number.py      # Numeric data types
│   ├── secs_data_str.py         # String data type
│   ├── secs_data_a.py           # Array data type
│   ├── secs_data_b.py           # Binary data type
│   ├── secs_data_j.py           # JSON data type
│   ├── secs_data_l.py           # List data type
│   ├── secsgem_package_base.py  # Base package implementation
│   ├── secsgem_package_0_1.py   # SECS/GEM v0.1 implementation
│   ├── sml_parser.py            # SML parser
│   ├── secs_script.py           # Script processing
│   └── script_runner.py         # Script execution engine
├── tests/                       # Test suite
├── pyproject.toml              # Poetry configuration
├── Dockerfile                  # Docker container definition
├── tox.ini                     # Test automation configuration
└── README.md                   # This file
```

## Data Types

The simulator supports the following SECS data types:

- **Boolean**: Boolean values (TRUE/FALSE)
- **Number**: Numeric values (I1, I2, I4, I8, F4, F8, U1, U2, U4, U8)
- **String**: Text data (ASCII, Unicode)
- **Array**: Ordered collections (A)
- **Binary**: Raw binary data (B)
- **JSON**: JSON formatted data (J)
- **List**: Nested lists (L)

## Dependencies

### Runtime Dependencies

- `python >= 3.7`
- `lupa >= 1.13` - Lua scripting engine
- `appdirs >= 1.4.4` - Application directory management

### Development Dependencies

- `pytest >= 7.1` - Testing framework
- `prospector >= 1.7.7` - Code quality checks
- `radon >= 5.1.0` - Code analysis
- `tox >= 3.25.1` - Test automation

## Development

### Running Tests

```bash
# Using uv
uv run pytest

# Using tox
uv run tox
```

### Code Quality Checks

```bash
# Run prospector
uv run prospector

# Run radon for complexity analysis
uv run radon cc secsgem_simulator/
```

## Network Ports

- **4000**: Default command server port
- **5000**: Secondary communication port (configurable)

## License

Please refer to the LICENSE file for details.

## Author

Benjamin Parzella <bparzella@gmail.com>

## Contributing

Contributions are welcome! Please ensure:
- Code follows PEP 8 style guidelines
- All tests pass before submitting
- New features include appropriate tests
- Documentation is updated accordingly

## Support

For issues, questions, or contributions, please visit the project repository.

