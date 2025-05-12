# Integration of @flamework/core, @rbxts/react, and @rbxts/charm

This integration combines Flamework's structured controller system with React's UI components and Charm's state management to create a robust application architecture for Roblox TypeScript projects.

## Required Packages

For full integration, you'll need these packages:

- @flamework/core - For controller, service, and system structure
- @rbxts/react - For React-like UI components
- @rbxts/react-roblox - For React portal rendering
- @rbxts/charm - For state management
- @rbxts/flamework-react-utils - For connecting Flamework with React
- @rbxts/react-charm - For using Charm atoms with React hooks

## Controller-Based State Management

Flamework controllers serve as state containers and business logic handlers:

```ts
import { Controller, OnStart, OnInit } from "@flamework/core";
import { atom } from "@rbxts/charm";
import { useAtom } from "@rbxts/react-charm";

@Controller({})
export class ShopMenuController implements OnStart, OnInit {
  private readonly _isMenuVisibleAtom = atom(false);
  private readonly _otherValueAtom = atom(0); // Example of another atom

  onInit() {
    // Initialization logic
  }

  onStart() {
    // Setup logic after dependencies are available
  }

  // Get atom value from outside UI
  public GetMenuVisibleAtom() {
    return this._isMenuVisibleAtom();
  }

  // Set atom value from anywhere
  public SetMenuVisible(value: boolean) {
    this._isMenuVisibleAtom(value);
  }

  // React hook for components
  public useMenuVisibleAtom() {
    return useAtom(this._isMenuVisibleAtom);
  }
}
```

## React Components Using Controller State

React components can access controller state through custom hooks:

```tsx
import React from "@rbxts/react";
import { useFlameworkDependency } from "@rbxts/flamework-react-utils";
import { ShopMenuController } from "path/to/controller";

function ShopMenu() {
  // Access controller with memoization
  const shopMenuController = useFlameworkDependency<ShopMenuController>();

  // Use the controller's atom hook
  const isVisible = shopMenuController.useMenuVisibleAtom();

  return (

       shopMenuController.SetMenuVisible(!isVisible)
        }}
      />
      {isVisible && (

          {/* Shop content */}

      )}

  );
}
```

## UI Initialization in Flamework Lifecycle

UI should be initialized within a Flamework controller to ensure proper dependency injection:

```ts
import { Controller, OnStart, OnInit } from "@flamework/core";
import { Players } from "@rbxts/services";
import React from "@rbxts/react";
import { createRoot, createPortal } from "@rbxts/react-roblox";
import { App } from "path/to/app";

@Controller({})
export class UIController implements OnStart, OnInit {
  onInit() {
    // Pre-dependency initialization
  }

  onStart() {
    // Initialize React after Flamework has set up all controllers
    const playerGui = Players.LocalPlayer.WaitForChild("PlayerGui");
    const container = new Instance("Folder");
    container.Name = "ReactRoot";
    container.Parent = playerGui;

    // Create React root in the Controller to ensure proper Flamework lifecycle integration
    const root = createRoot(container);
    root.render(

        {createPortal(, playerGui)}

    );
  }
}
```

## Best Practices

1. **Controller Responsibility**: Controllers should own and manage atoms, providing accessor methods and hooks for UI components.

2. **Component Access**: Always use `useFlameworkDependency` instead of direct `Dependency` calls in React components for proper memoization.

3. **UI Initialization**: Always initialize React within a Flamework controller's `onStart` method to ensure all dependencies are properly set up.

4. **Atom Naming**: Follow the convention of adding "Atom" as a suffix to atom variables and functions that can be subscribed to.

5. **Error Handling**: Add appropriate error handling when accessing services or when UI components depend on data that might not be immediately available.

6. **Side Effects**: Use Charm's `effect` utility for handling side effects outside of React components:

```ts
import { effect } from "@rbxts/charm";

@Controller({})
export class GameStateController {
  private readonly _gameStateAtom = atom("lobby");

  onStart() {
    // React to state changes outside of UI components
    effect(() => {
      const gameState = this._gameStateAtom();
      print(`Game state changed to: ${gameState}`);

      // Return cleanup function if needed
      return () => {
        print("Cleaning up previous state");
      };
    });
  }
}
```

This integration pattern creates a clean separation of concerns: Flamework handles game structure and business logic, React manages UI rendering, and Charm provides reactive state management that works across both systems.
