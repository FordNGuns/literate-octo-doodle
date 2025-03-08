# PlayerManager Module Documentation

This module manages player profiles, handles data synchronization, and provides utility functions for interacting with player data.

---

## `PlayerManager.AddPlayer(player: Player)`

### Description
This function loads and initializes a player's profile when they join the game. It creates a profile using `ProfileService`, sets up a replica for data synchronization, and initializes the player's leaderboard stats.

### Parameters
- `player` (`Player`): The player object representing the player who joined the game.

### Behavior
1. Attempts to load the player's profile using `ProfileService`.
   - If the game is running in Roblox Studio, it uses a mock profile.
   - Otherwise, it loads the profile from the live data store.
2. If the profile is successfully loaded:
   - Adds the player's `UserId` to the profile.
   - Reconciles the profile to ensure data consistency.
   - Sets up a listener to handle profile release (e.g., when the player leaves or is kicked).
   - Creates a `PlayerProfile` object containing the profile, replica, and player reference.
   - Stores the `PlayerProfile` in `PlayerManager.__Profiles`.
   - Adds the player's name to the `PlayerManager.__Loaded` table.
   - Initializes the player's leaderboard stats (e.g., `Level`).
3. If the profile fails to load, the player is kicked from the game.

### Example
```lua
Players.PlayerAdded:Connect(function(player)
    PlayerManager.AddPlayer(player)
end)
```

---

## `PlayerManager.RemovePlayer(player: Player)`

### Description
This function handles the cleanup when a player leaves the game. It releases the player's profile and removes their data from the `PlayerManager` tables.

### Parameters
- `player` (`Player`): The player object representing the player who left the game.

### Behavior
1. Retrieves the player's profile from `PlayerManager.__Profiles`.
2. If the profile exists:
   - Releases the profile, which triggers the `ListenToRelease` callback (if defined).
3. Removes the player's name from the `PlayerManager.__Loaded` table.

### Example
```lua
Players.PlayerRemoving:Connect(function(player)
    PlayerManager.RemovePlayer(player)
end)
```

---

## `PlayerManager.GetPlayerProfile(player: Player)`

### Description
This function retrieves the `PlayerProfile` object associated with a player. It can accept either a `Player` object or a player's name as input.

### Parameters
- `player` (`Player` or `string`): The player object or the name of the player whose profile is being retrieved.

### Returns
- The `PlayerProfile` object associated with the player, or `nil` if the player does not have a profile.

### Example
```lua
local profile = PlayerManager.GetPlayerProfile(player)
if profile then
    print("Player Level:", profile.Replica.Data.Level)
end
```
---

## `PlayerManager.IsActive(playerName: string)`

### Description
This function checks if a player has an active profile and whether their profile is fully loaded.

### Parameters
- `playerName` (`string`): The name of the player to check.

### Returns
- `hasProfile` (`boolean`): `true` if the player has a profile, otherwise `false`.
- `isLoaded` (`boolean`): `true` if the player's profile is fully loaded, otherwise `false`.

### Example
```lua
local hasProfile, isLoaded = PlayerManager.IsActive("Player1")
if hasProfile and isLoaded then
    print("Player1 is fully loaded and active.")
end
```

---

## Notes

### `PlayerProfile` Object
The `PlayerProfile` object contains the following fields:
- `Profile`: The player's data profile loaded from `ProfileService`.
- `Replica`: The replica object used for data synchronization.
- `_player`: A reference to the player object.

### Tables
- `PlayerManager.__Profiles`: Stores all active player profiles, indexed by player name.
- `PlayerManager.__Loaded`: Tracks which players have fully loaded profiles.

### Dependencies
- `ProfileService`: Used for loading and managing player profiles.
- `ReplicaService`: Used for creating and managing data replicas.
- `PlayerProfile`: A module that defines the structure and behavior of a player's profile.

# `PlayerManager.__Profiles[Player.Name]` Functions

The `PlayerManager.__Profiles[Player.Name]` object is an instance of the `PlayerProfile` class, which provides methods to interact with a player's profile data. Below are the functions available for this object:

---

## `PlayerProfile:UpdateXP(value: number)`

### Description
Adds the specified amount of experience points (XP) to the player's total accumulated XP. If the player's XP meets or exceeds the required amount for the next level, the player's level is incremented, and the XP is reset accordingly.

### Parameters
- `value` (`number`): The amount of XP to add to the player's current XP.

### Behavior
1. Updates the player's XP in the replica data.
2. Calculates the XP required to level up using the formula:
   ```
   xpNeeded = math.floor((level * 70) * (level + 10) / 6 - 0.01 * level)
   ```
3. If the player's XP meets or exceeds `xpNeeded`:
   - Increments the player's level in the leaderboard.
   - Updates the player's level in the replica data.
   - Resets the player's XP by subtracting `xpNeeded`.

### Example
```lua
local profile = PlayerManager.GetPlayerProfile(player)
profile:UpdateXP(100) -- Adds 100 XP to the player's profile
```

---

## `PlayerProfile:AddCoins(value: number)`

### Description
Adds the specified amount of coins to the player's total coins.

### Parameters
- `value` (`number`): The amount of coins to add to the player's current coins.

### Behavior
1. Updates the player's coins in the replica data by adding the specified value.

### Example
```lua
local profile = PlayerManager.GetPlayerProfile(player)
profile:AddCoins(50) -- Adds 50 coins to the player's profile
```

---

## `PlayerProfile:RemoveCoins(value: number)`

### Description
Removes the specified amount of coins from the player's total coins.

### Parameters
- `value` (`number`): The amount of coins to remove from the player's current coins.

### Behavior
1. Updates the player's coins in the replica data by subtracting the specified value.

### Example
```lua
local profile = PlayerManager.GetPlayerProfile(player)
profile:RemoveCoins(20) -- Removes 20 coins from the player's profile
```

---

## Notes

### Replica Data
The `PlayerProfile` object interacts with the player's data through the `Replica` object. The following fields are used:
- `XP`: The player's current experience points.
- `Level`: The player's current level.
- `Coins`: The player's current coin balance.

### Leaderboard Integration
The player's level is also reflected in the `leaderstats` folder, which is visible in the Roblox leaderboard.

### Example Usage
```lua
-- Example: Grant XP and coins to a player
local profile = PlayerManager.GetPlayerProfile(player)
if profile then
    profile:UpdateXP(150) -- Add 150 XP
    profile:AddCoins(100) -- Add 100 coins
end
```
