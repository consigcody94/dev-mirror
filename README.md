# Dev Mirror

A Model Context Protocol (MCP) server for tracking developer productivity metrics and comparing AI-assisted vs manual coding sessions.

## Overview

Dev Mirror helps you answer the question: "Is AI actually making me more productive?" Rather than relying on subjective feelings, this tool tracks objective metrics across your development sessions to provide data-driven insights.

### The Problem

Studies suggest developers using AI tools may be "19% slower" - but raw speed metrics miss the bigger picture. What matters is:

- Did the code work on the first try?
- How many iterations did it take to get a working solution?
- What was the build success rate?
- How often did you context switch?

### The Solution

Dev Mirror tracks what actually matters - your real productivity metrics across AI-assisted and manual coding sessions, then generates comparative reports.

## Features

- **Session Tracking** - Start, update, and end development sessions with full metrics
- **Statistics Dashboard** - View aggregated stats filtered by type, time range, and tags
- **AI vs Manual Comparison** - Direct side-by-side comparison of development approaches
- **Quality Scoring** - Objective code quality scores based on multiple factors
- **Report Generation** - Comprehensive Markdown or JSON reports

## Installation

```bash
# Clone the repository
git clone https://github.com/consigcody94/dev-mirror.git
cd dev-mirror

# Install dependencies
npm install

# Build the project
npm run build
```

## Configuration

### Data Storage

By default, session data is stored in `~/.dev-mirror/sessions.json`. You can customize this location:

```bash
export DEV_MIRROR_DATA_DIR="/path/to/custom/data/directory"
```

### Claude Desktop Integration

Add to your `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "dev-mirror": {
      "command": "node",
      "args": ["/absolute/path/to/dev-mirror/build/index.js"],
      "env": {
        "DEV_MIRROR_DATA_DIR": "/optional/custom/path"
      }
    }
  }
}
```

**Config file locations:**
- macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`
- Windows: `%APPDATA%\Claude\claude_desktop_config.json`
- Linux: `~/.config/Claude/claude_desktop_config.json`

## Tools Reference

### track_session

Start, update, or end a development session.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `action` | string | Yes | `"start"`, `"end"`, or `"update"` |
| `sessionId` | string | For end/update | Session ID returned from start |
| `type` | string | No | `"ai-assisted"` or `"manual"` (default: ai-assisted) |
| `task` | string | No | Description of the task |
| `linesAdded` | number | No | Lines of code added |
| `linesDeleted` | number | No | Lines of code deleted |
| `filesModified` | number | No | Number of files changed |
| `testsPassed` | number | No | Number of passing tests |
| `testsFailed` | number | No | Number of failing tests |
| `buildSucceeded` | boolean | No | Whether the build passed |
| `iterationCount` | number | No | Number of attempts/iterations |
| `contextSwitches` | number | No | Times you switched tasks |
| `notes` | string | No | Additional notes |
| `tags` | string[] | No | Tags for categorization |

**Example - Start a session:**

```json
{
  "action": "start",
  "type": "ai-assisted",
  "task": "Implement user authentication",
  "tags": ["backend", "security"]
}
```

**Example - End a session:**

```json
{
  "action": "end",
  "sessionId": "session-1234567890",
  "linesAdded": 250,
  "linesDeleted": 30,
  "filesModified": 5,
  "testsPassed": 15,
  "testsFailed": 0,
  "buildSucceeded": true,
  "iterationCount": 2,
  "contextSwitches": 1
}
```

### get_stats

Retrieve aggregated productivity statistics.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `type` | string | No | `"all"`, `"ai-assisted"`, or `"manual"` (default: all) |
| `timeRange` | string | No | `"day"`, `"week"`, `"month"`, or `"all"` (default: all) |
| `tags` | string[] | No | Filter by specific tags |

**Example:**

```json
{
  "type": "ai-assisted",
  "timeRange": "week",
  "tags": ["backend"]
}
```

**Response includes:**
- Total sessions count
- AI-assisted vs manual session counts
- Total time in minutes
- Average session duration
- Total lines added/deleted
- Build success rate
- Average iterations per session
- Average context switches

### compare_ai_vs_manual

Generate a direct comparison between AI-assisted and manual development.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `timeRange` | string | No | `"day"`, `"week"`, `"month"`, or `"all"` |
| `metrics` | string[] | No | Specific metrics to compare |

**Available metrics:**
- `duration` - Average session duration
- `linesPerMinute` - Code output rate
- `testPassRate` - Percentage of tests passing
- `buildSuccessRate` - Percentage of successful builds
- `iterationsPerSession` - Average attempts needed
- `contextSwitches` - Average task switches

**Example:**

```json
{
  "timeRange": "month",
  "metrics": ["duration", "buildSuccessRate", "iterationsPerSession"]
}
```

### code_quality_score

Calculate a quality score (0-100) for sessions.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `sessionId` | string | No | Specific session to score (omit for all) |
| `weights` | object | No | Custom scoring weights |

**Default weights:**
- Test Pass Rate: 30%
- Build Success: 30%
- Low Iterations: 20% (fewer = better)
- Low Context Switches: 20% (fewer = better)

**Example with custom weights:**

```json
{
  "weights": {
    "testPassRate": 0.4,
    "buildSuccess": 0.3,
    "lowIterations": 0.2,
    "lowContextSwitches": 0.1
  }
}
```

### generate_report

Generate a comprehensive productivity report.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `format` | string | No | `"markdown"` or `"json"` (default: markdown) |
| `timeRange` | string | No | `"day"`, `"week"`, `"month"`, or `"all"` (default: week) |
| `includeCharts` | boolean | No | Include ASCII charts (default: true) |

**Example:**

```json
{
  "format": "markdown",
  "timeRange": "week"
}
```

## Workflow Example

1. **Start your coding session:**
   ```
   track_session with action: "start", type: "ai-assisted", task: "Add payment processing"
   ```

2. **Code your feature** (with or without AI assistance)

3. **End the session with metrics:**
   ```
   track_session with action: "end", sessionId: "...", linesAdded: 150, testsPassed: 8, buildSucceeded: true
   ```

4. **Review your productivity:**
   ```
   compare_ai_vs_manual with timeRange: "week"
   ```

5. **Generate a report:**
   ```
   generate_report with format: "markdown", timeRange: "month"
   ```

## Data Format

Sessions are stored as JSON in `~/.dev-mirror/sessions.json`:

```json
[
  {
    "id": "session-1234567890",
    "startTime": "2024-01-15T10:00:00.000Z",
    "endTime": "2024-01-15T11:30:00.000Z",
    "type": "ai-assisted",
    "task": "Implement user authentication",
    "linesAdded": 250,
    "linesDeleted": 30,
    "filesModified": 5,
    "testsPassed": 15,
    "testsFailed": 0,
    "buildSucceeded": true,
    "iterationCount": 2,
    "contextSwitches": 1,
    "notes": "Used Claude for boilerplate",
    "tags": ["backend", "security"]
  }
]
```

## Requirements

- Node.js 18 or higher
- npm or yarn

## Troubleshooting

### Server won't start

1. Ensure Node.js 18+ is installed: `node --version`
2. Rebuild the project: `npm run build`
3. Check the path in your Claude Desktop config is absolute

### Data not persisting

1. Check write permissions to the data directory
2. Verify `DEV_MIRROR_DATA_DIR` is set correctly if using custom location

### Sessions not showing in stats

1. Ensure sessions are properly ended with `action: "end"`
2. Check the `timeRange` filter isn't excluding your sessions

## License

MIT License - see [LICENSE](LICENSE) file for details.

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## Author

consigcody94
