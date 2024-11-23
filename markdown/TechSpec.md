# Helicopter Game Technical Specification - Phaser.io Implementation

## Game Architecture

### 1. Game Configuration
```typescript
const config = {
    type: Phaser.AUTO,          // Renderer type (auto-selects WebGL or Canvas)
    width: 800,                 // Game window width in pixels
    height: 600,                // Game window height in pixels
    physics: {
        default: 'arcade',      // Physics engine selection
        arcade: {
            gravity: { y: 300 }, // Vertical gravity force
            debug: false        // Physics debug rendering flag
        }
    },
    scene: [MenuScene, GameScene, GameOverScene] // Game scene sequence
}
```

### 2. Scenes

#### MenuScene (extends Phaser.Scene)
Main menu screen controller

**Properties:**
- leaderboard: LeaderboardManager  // Handles high score data and display

**Methods:**
- preload()
  - Behavior: Loads all required assets for menu
  - Output: Loaded assets in scene cache
- create()
  - Behavior: Sets up menu UI elements and event listeners
  - Output: Initialized menu interface
- startGame()
  - Behavior: Transitions to game scene
  - Output: Launches GameScene

#### GameScene (extends Phaser.Scene)
Main gameplay controller

**Properties:**
- helicopter: HelicopterSprite    // Player-controlled entity
- enemies: Phaser.GameObjects.Group // Collection of active enemies
- pickups: Phaser.GameObjects.Group // Collection of active collectibles
- levelManager: LevelManager      // Controls level generation and progression
- score: number                   // Current player score

**Methods:**
- preload()
  - Behavior: Loads game assets
  - Output: Loaded game assets in scene cache
- create()
  - Behavior: Initializes game world, physics, and starting entities
  - Output: Ready game world
- update(time: number, delta: number)
  - Behavior: Updates game state each frame
  - Input: Current time, time since last frame
  - Output: Updated game state
- handleCollisions()
  - Behavior: Processes all entity collisions
  - Output: Updated entity states post-collision
- spawnPickup(type: PickupType, x: number, y: number)
  - Behavior: Creates new pickup at specified location
  - Input: Pickup type, spawn coordinates
  - Output: New pickup entity

### 3. Game Objects

#### HelicopterSprite (extends Phaser.Physics.Arcade.Sprite)
Player character controller

**Properties:**
- fuel: number                    // Current fuel amount (0-100)
- health: number                  // Current health points (0-100)
- ammunition: number              // Available weapon ammunition
- turret: Phaser.GameObjects.Sprite // Weapon attachment sprite

**Methods:**
- create()
  - Behavior: Initializes helicopter sprite and physics body
  - Output: Ready helicopter entity
- update()
  - Behavior: Processes helicopter state each frame
  - Output: Updated helicopter position and state
- handleInput()
  - Behavior: Processes player input
  - Output: Updated helicopter controls state
- applyThrust()
  - Behavior: Applies upward force when 'W' pressed
  - Output: Updated velocity and fuel consumption
- rotateTurret()
  - Behavior: Aims turret toward mouse position
  - Input: Mouse coordinates
  - Output: Updated turret rotation
- fire()
  - Behavior: Spawns projectile in turret direction
  - Output: New projectile entity, reduced ammunition
- takeDamage(amount: number)
  - Behavior: Reduces helicopter health
  - Input: Damage amount
  - Output: Updated health value
- collectPickup(type: PickupType, amount: number)
  - Behavior: Processes resource collection
  - Input: Pickup type, resource amount
  - Output: Updated resource values

#### Enemy (extends Phaser.Physics.Arcade.Sprite)
Enemy unit controller

**Properties:**
- health: number                  // Enemy hit points
- attackTimer: Phaser.Time.TimerEvent // Controls attack frequency

**Methods:**
- update()
  - Behavior: Updates enemy AI and state
  - Output: Updated enemy behavior
- attack()
  - Behavior: Initiates enemy attack sequence
  - Output: Spawned projectile
- takeDamage(amount: number)
  - Behavior: Processes enemy damage
  - Input: Damage amount
  - Output: Updated health, death if zero

### 4. Managers

#### LevelManager
Level generation and control

**Properties:**
- currentLevel: number            // Active level index
- scene: GameScene               // Reference to main game scene
- platforms: Phaser.GameObjects.Group // Collection of level obstacles

**Methods:**
- generateLevel(difficulty: number)
  - Behavior: Creates new level layout
  - Input: Difficulty multiplier
  - Output: Generated level structure
- createPlatform(x: number, y: number, width: number)
  - Behavior: Spawns platform obstacle
  - Input: Platform dimensions and position
  - Output: New platform object
- spawnEnemies()
  - Behavior: Places enemies in level
  - Output: New enemy entities
- spawnPickups()
  - Behavior: Distributes resource pickups
  - Output: New pickup entities
- checkLevelComplete()
  - Behavior: Verifies level completion
  - Output: Boolean completion status

### 5. UI Components

#### HUD (extends Phaser.GameObjects.Container)
Game interface display

**Properties:**
- healthBar: ProgressBar          // Health indicator
- fuelBar: ProgressBar           // Fuel level indicator
- ammoText: Phaser.GameObjects.Text // Ammunition counter
- scoreText: Phaser.GameObjects.Text // Score display

**Methods:**
- create()
  - Behavior: Initializes HUD elements
  - Output: Ready UI display
- updateHealth(value: number)
  - Behavior: Updates health display
  - Input: New health value
  - Output: Updated health bar
- updateFuel(value: number)
  - Behavior: Updates fuel display
  - Input: New fuel value
  - Output: Updated fuel bar
- updateAmmo(value: number)
  - Behavior: Updates ammunition counter
  - Input: New ammo count
  - Output: Updated ammo display
- updateScore(value: number)
  - Behavior: Updates score counter
  - Input: New score value
  - Output: Updated score display

#### Progress Bar Class (extends Phaser.GameObjects.Container)
A flexible progress bar component for displaying resource levels (health, fuel, ammunition) in the game UI.

| background | Phaser.GameObjects.Rectangle | Background rectangle with white stroke |
| bar | Phaser.GameObjects.Rectangle | Foreground rectangle showing progress |
| label | Phaser.GameObjects.Text | Resource name display |
| value | number | Current resource value (default: 100) |
| maxValue | number | Maximum possible value (default: 100) |

**Methods:**
- setValue(amount: number)

**Purpose**: Update current value

**Parameters**:
- `amount`: New value for the resource

**Behavior**:
- Clamps input between 0 and maxValue
- Updates bar scale
- Triggers color update if threshold crossed

**Returns**: None (visual update only)

#### setMaxValue

```typescript
setMaxValue(amount: number): void
```

**Purpose**: Update maximum value

**Parameters**:
- `amount`: New maximum value

**Behavior**:
- Updates maximum value
- Clamps current value if necessary
- Updates display

**Returns**: None (visual update only)

#### getCurrentValue

```typescript
getCurrentValue(): number
```

**Purpose**: Get current value

**Parameters**: None

**Returns**: Current resource value

#### Private updateDisplay

```typescript
private updateDisplay(): void
```

**Purpose**: Update visual representation

**Behavior**:
- Calculates percentage (value/maxValue)
- Updates bar scale
- Updates color based on thresholds:
  - ≤ 20%: Red
  - ≤ 50%: Yellow
  - > 50%: Original color

**Returns**: None (visual update only)

## Leaderboard Manager Class

Manages high score data storage, retrieval, and display using Firebase Firestore.

### Class Definition

```typescript
class LeaderboardManager
```

### Properties

#### Private Properties

| firestore | firebase.firestore.Firestore | Firebase Firestore instance |
| collectionName | string | Firestore collection identifier (default: 'highscores') |
| cachedScores | Array<{name: string, score: number}> | Local cache of high scores |
| maxEntries | number | Maximum leaderboard entries (default: 100) |

### Methods

#### Constructor

```typescript
constructor(firebaseApp: firebase.app.App)
```

**Purpose**: Initialize leaderboard manager

**Parameters**:
- `firebaseApp`: Initialized Firebase app instance

**Returns**: LeaderboardManager instance

#### submitScore

```typescript
async submitScore(playerName: string, score: number): Promise<boolean>
```

**Purpose**: Submit new high score

**Parameters**:
- `playerName`: Player identifier
- `score`: Achieved score

**Behavior**:
- Checks if score qualifies for leaderboard
- Adds score to database if qualified
- Includes timestamp
- Invalidates cache

**Returns**: Promise resolving to submission success

#### getTopScores

```typescript
async getTopScores(limit: number = 10): Promise<Array<{name: string, score: number}>>
```

**Purpose**: Retrieve top scores

**Parameters**:
- `limit`: Number of scores to retrieve (default: 10)

**Behavior**:
- Returns cached scores if available
- Queries database if cache empty
- Orders scores descending
- Updates cache with results

**Returns**: Promise resolving to array of score objects

#### getPlayerRank

```typescript
async getPlayerRank(score: number): Promise<number>
```

**Purpose**: Calculate player ranking

**Parameters**:
- `score`: Player's score

**Behavior**:
- Counts scores higher than input
- Adds 1 for 1-based ranking

**Returns**: Promise resolving to player's rank (1-based)

#### createLeaderboardDisplay

```typescript
createLeaderboardDisplay(
  scene: Phaser.Scene, 
  x: number, 
  y: number
): Phaser.GameObjects.Container
```

**Purpose**: Create visual leaderboard

**Parameters**:
- `scene`: Active Phaser scene
- `x, y`: Position coordinates

**Behavior**:
- Creates container
- Adds title text
- Retrieves and displays top scores
- Formats each score entry

**Returns**: Container with leaderboard display

### Data Structures

#### Score Entry Interface

```typescript
interface ScoreEntry {
    name: string;      // Player identifier
    score: number;     // Numeric score
    timestamp?: Date;  // Submission time
}
```

## Implementation Notes

1. **Key Event Flows**
   - Player Input → handleInput() → applyThrust()/rotateTurret() → Physics Update
   - Collision → handleCollisions() → takeDamage()/collectPickup() → UI Update
   - Enemy AI → update() → attack() → Projectile Spawn

2. **Resource Management**
   - Fuel decreases continuously during thrust
   - Health only changes on collision/damage
   - Ammunition decreases per shot
   - All resources replenished by respective pickups

3. **Level Progression**
   - Difficulty increases with level number
   - Enemy count and aggression scales with difficulty
   - Pickup frequency adjusts inversely with difficulty

4. **Performance Considerations**
   - Use object pooling for frequent entities (projectiles, particles)
   - Implement camera culling for off-screen entities
   - Cleanup inactive entities and event listeners

## Figma Link
 - https://www.figma.com/board/w3wphcosf5TgLn68LxQMfD/Fat-Helicopter-Tech-Spec?node-id=0-1&t=e5mSw3EQxbmqaYce-1