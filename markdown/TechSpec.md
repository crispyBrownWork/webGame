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