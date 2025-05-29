# HoopyGameFramework Analysis

## 1. Overall Summary
   - **Brief overview of the framework**: HoopyGameFramework is a Unity-based framework designed to accelerate game development by providing a structured architecture, common game systems, and editor productivity tools. It emphasizes modularity, testability, and efficient asset management.
   - **Key Goals & Design Philosophy**: The framework aims to provide a robust foundation for various game genres, enabling developers to focus on gameplay rather than boilerplate code. It follows an MVC-like pattern, promotes event-driven programming, and integrates several popular third-party plugins for enhanced functionality.
   - **Core Strengths**:
     - **Modularity**: Clearly defined layers for architecture, managers, UI, and editor tools.
     - **Extensibility**: Designed to be easily extended with new features and systems.
     - **Productivity**: Numerous editor tools automate common tasks like asset importing, script creation, and build processes.
     - **Hot Updates**: Built-in support for hot code and asset updates using YooAsset and custom tools.
     - **Pre-built Systems**: Includes common game features like audio management, object pooling, UI management, and scene management.
   - **Intended Use Cases**: Suitable for a wide range of Unity projects, from simple mobile games to more complex applications requiring robust architecture and efficient workflows. Particularly beneficial for teams looking for a standardized development approach and tools to streamline production.

## 2. Core Architecture (`Runtime/ArchitectureCore`)
   - **HGArchitecture**: This is the heart of the framework, acting as a central service locator or facade. It provides access to Models (data containers), Systems (logic controllers), and Utilities (helper classes). This promotes a clear separation of concerns and makes it easy to manage dependencies.
   - **MVC-like Pattern**: The framework implements a variation of the Model-View-Controller pattern.
     - **Models**: Store application state and data.
     - **Systems**: Contain game logic and act similarly to Controllers, manipulating Models and interacting with Views (UI).
     - **Views**: Primarily handled by the UI Framework, which displays data from Models and sends user input to Systems.
   - **Commands and Queries**: The framework utilizes Command and Query patterns for managing operations. Commands encapsulate actions that modify the application state (e.g., `AttackCommand`), while Queries retrieve data without altering the state (e.g., `GetPlayerHealthQuery`). This promotes a clean and testable way to handle game logic.
   - **IOC Container**: HoopyGameFramework leverages a custom `IOCContainer` for basic dependency injection and also integrates `VContainer`. This allows for decoupling of components, making the codebase more flexible and easier to test. Dependencies are registered and resolved through these containers.
   - **Event System**:
     - `Event.cs`: A simple, generic event class likely used for custom event definitions within specific systems.
     - `TypeEventSystem`: A more robust, type-based event system that allows for registering and triggering events based on their C# type. This provides strong typing and reduces the risk of errors associated with string-based events.
   - **Bindable Properties (`BindableProperty.cs`)**: These are observable properties that automatically notify listeners when their value changes. This is crucial for UI updates and reactive programming, allowing UI elements to bind directly to data and update automatically.

## 3. Managers (`Runtime/Managers`)
   - **AssetManager (`AssetMgr.cs`, `HotAssetConfig.cs`)**: This manager is built on top of the **YooAsset** plugin. It handles all aspects of asset loading, including synchronous and asynchronous operations. It supports loading individual assets, sub-assets (e.g., sprites from a sprite sheet), and entire scenes. A key feature is its support for hot updates, allowing game assets to be updated without requiring a new build of the application, configured via `HotAssetConfig.cs`.
   - **AudioManager (`AudioMgr.cs`, `AudioConfig.cs`)**: Manages background music (BGM) and sound effects (SFX). It utilizes Unity's `AudioMixer` for advanced audio control (e.g., volume groups, effects) and custom controllers for playback logic. Configurations such as volume levels, audio clips, and mixer groups are likely defined in `AudioConfig.cs`.
   - **EventManager (`EventMgr.cs`)**: Provides a global, string-based event system. This allows different parts of the game to communicate without direct dependencies. It supports events with no parameters and events with a single parameter, facilitating decoupled communication between game modules.
   - **ObjectPoolManager (`ObjectPoolMgr.cs`)**: Implements object pooling for GameObjects. This is a performance optimization technique that reuses frequently created and destroyed objects (like bullets or particle effects) instead of instantiating and destroying them, which can cause garbage collection spikes.
   - **SceneManager (`SceneMgr.cs`)**: Responsible for loading and unloading scenes. It supports both synchronous and asynchronous scene loading. A notable feature is its integrated loading UI, which can display progress and messages to the player during scene transitions.
   - **UIManager (`UIMgr.cs`)**: Manages the entire lifecycle of UI elements, which are categorized into Panels (full-screen interfaces) and Popups (modal dialogs or temporary messages). It uses `UILoador` (which in turn uses `AssetMgr`) to load UI prefabs. UI elements derive from `BaseUI` (and its specializations `BasePanel`, `BasePopup`). The manager also incorporates an LRU (Least Recently Used) cache (`LeastResentlyUsedUtility.cs`) to automatically unload UI elements that haven't been used recently, optimizing memory usage.

## 4. UI Framework (`Runtime/UIFramework`)
   - **Base Classes (`BaseUI.cs`, `BasePanel.cs`, `BasePopup.cs`)**: These abstract classes provide the foundational structure and lifecycle methods (e.g., `OnInit`, `OnShow`, `OnHide`, `OnClose`) for all UI elements. `BasePanel` typically represents full-screen UIs, while `BasePopup` is used for modal dialogs or temporary notifications. Developers create specific UI elements by inheriting from these classes.
   - **LRU Utility (`LeastResentlyUsedUtility.cs`)**: This utility implements a Least Recently Used caching strategy for UI elements. When the UI manager needs to free up memory or reduce the number of active UI objects, it can use this utility to identify and destroy the UI elements that haven't been accessed for the longest time.
   - **UI Loader (`UILoador.cs`)**: This class is responsible for loading UI prefabs from asset bundles or the Resources folder, utilizing the `AssetMgr`. It handles the instantiation of UI GameObjects and prepares them for use by the `UIMgr`.
   - **Data Handling (`IUIDataBase.cs`, `UIType.cs`)**:
     - `IUIDataBase.cs`: An interface likely used to define a contract for data objects that are passed to UI elements when they are opened or updated. This ensures that UI elements receive data in a structured way.
     - `UIType.cs`: An enumeration or a class containing constants that define unique identifiers or types for different UI elements. This is used by the `UIMgr` to manage and retrieve specific UIs.

## 5. Editor Tools (`Editor/`)
   - **Custom Build Pipeline (`CustomBuildPipelineEditor.cs`)**: Automates the process of creating game builds for different platforms (e.g., Android, iOS, Windows). It likely handles steps like scene selection, platform switching, setting build options, and signing, streamlining the build process.
   - **DLL to Bytes Converter (`DLL2BytesFileEditor.cs`)**: A tool used in the hot update process. It converts compiled C# assemblies (DLLs) into `.bytes` files. These byte arrays can then be loaded at runtime, allowing for hot-swapping of game logic without a full application update.
   - **GitHub Package Importer (`ImportPackageEditor.cs`)**: Provides a simple editor window to browse and open URLs, likely pointing to GitHub repositories for Unity packages or useful web-based documentation, enhancing developer workflow.
   - **Excel Importer (`ExcelImporter.cs`)**: Automatically imports data from Excel spreadsheets (`.xls`, `.xlsx`) into ScriptableObjects. It uses the NPOI library to read Excel files. This is extremely useful for game configuration, localization, and managing large datasets that can be easily edited by designers.
   - **Script Creator (`GeneratorCustomScriptFile.cs`)**: Generates new C# scripts based on predefined templates. This can save time and ensure consistency when creating new classes for the framework (e.g., new Systems, Models, UI Panels).
   - **Preset Importer (`PresetImportPerFolder.cs`)**: Applies Unity Presets to assets automatically based on the folder they are imported into. This helps maintain consistency in asset import settings (e.g., texture compression, model import settings) across the project.
   - **Utility Tools**:
     - **Copy Object Path**: An editor utility to copy the full hierarchy path of a GameObject in the scene, useful for debugging or scripting.
     - **Remove Missing Scripts**: A tool to find and remove script components from GameObjects where the underlying script file is missing, which can cause errors and warnings.
   - **Scoped Registry Helper (`ScopedRegistryHelper.cs`)**: Modifies the `Packages/manifest.json` file to add or update scoped registries, such as OpenUPM. This makes it easier to manage and discover third-party Unity packages.

## 6. Key Plugins (`Runtime/Plugins/`)
   - **DOTween**: A powerful and flexible tweening engine used for creating programmatic animations for UI elements, game objects, and properties. It's known for its performance and ease of use.
   - **UniTask**: An optimized async/await library for Unity that provides a more efficient alternative to standard C# Tasks, especially for Unity's single-threaded environment. It helps manage asynchronous operations without incurring performance overhead.
   - **VContainer**: A lightweight, high-performance dependency injection library. It's used alongside the custom IOC container to manage dependencies between different parts of the application, promoting loose coupling and testability.
   - **YooAsset**: A comprehensive asset management system for Unity. It handles asset loading, asset bundle management, resource updates (hot updates), and provides features like reference counting and asynchronous loading. It's central to the framework's asset management strategy.
   - **Luban**: A game data configuration solution. It processes Excel files (and other formats) into various runtime data structures (e.g., C# classes, JSON). This allows game designers to manage complex game data in spreadsheets, which are then transformed into an efficient runtime format.

## 7. Sample Features & Modules (`Samples~/`)
   - **Core Architecture & UI Framework Demos**: The samples provide practical examples of how to use the `HGArchitecture` (Models, Systems, Utilities) and the `UIFramework` (Panels, Popups, data binding). These serve as a starting point for developers to understand the framework's intended usage patterns.
   - **Editor Productivity Tools**:
     - **AutoBind**: A tool (likely an editor script) that automatically finds and assigns references to components within a UI prefab (e.g., Text, Button, Image components to corresponding C# script variables), reducing manual setup.
     - **ClickEffect**: A sample demonstrating how to easily add visual or auditory feedback to UI button clicks.
   - **Common Game Systems**:
     - **Quiz/Examine**: A module for creating quiz or examination-style gameplay, likely including data structures for questions/answers and UI for presentation.
     - **Gifts (Luck Spin, Daily Sign-in)**: Examples of common monetization or retention mechanics, showcasing how to implement systems like a spinning wheel for rewards or daily login bonuses.
     - **In-game Guides**: A system for displaying tutorials or contextual help to players within the game.
     - **Hot Update Panel**: A UI panel that manages the hot update process, showing progress and allowing the user to trigger updates. This works in conjunction with YooAsset and the DLL-to-bytes conversion.
     - **Red Point (Notification) System**: A common UI feature to indicate new content or pending actions (e.g., an exclamation mark on an inventory icon).
   - **Gameplay Mechanics**:
     - **Command Pattern**: Demonstrations of how to use the command pattern (as part of `HGArchitecture`) for handling player actions or game events.
     - **Player Controller Templates**: Basic templates or examples for player character controllers, possibly showing movement, input handling, or interactions within the game world.
