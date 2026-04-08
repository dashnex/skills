# CLI Commands

`@dashnex/cli` provides an extendable CLI tool. Each module can add its own commands.

## Creating a Command

### 1. Create command handler

```ts
// src/commands/version.ts
import { DashnexCliCommand } from "@dashnex/types";

class VersionCommand implements DashnexCliCommand {
  async execute(options: any) {
    const packageJson = await import("../../package.json");
    console.log(`Version: ${packageJson.version}`);
  }
}

export default VersionCommand;
```

### 2. Register in commands index

```ts
// src/commands/index.ts
import { DashnexCliCommandConfig } from "@dashnex/types";
import VersionCommand from "./version";

const commands: DashnexCliCommandConfig[] = [
  {
    name: "version",
    description: "Display the version",
    handler: new VersionCommand(),
    options: []
  }
];

export default commands;
```

### 3. Run

```bash
npx dashnex <command>
```

## Interfaces

```ts
export interface DashnexCliCommand {
  execute: (options: any) => Promise<void>;
}

export interface DashnexCliCommandConfig {
  name: string;
  description: string;
  options?: DashnexCliOption[];
  subcommands?: DashnexCliCommandConfig[];
  handler: DashnexCliCommand;
}
```
