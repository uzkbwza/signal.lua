# signal.lua

this is a simple and slightly opinionated but powerful lua library (and one of many) that implements the [observer](https://gameprogrammingpatterns.com/observer.html) pattern. vaguely influenced by Godot signals. use this to send messages from an object without having to care who listens.

## example usage
frequently, things in video games keep stored a number of "healths" so that they don't immediately die when they are born. sometimes you want a visual representation of how many healths a video game thing has on the screen, so you can know if it is close to dying. but it would be annoying if the thing had to remember to show its own healths meter. really it shouldnt care about that. 

### there is a better way

imagine it like this:
- the player is a radio deejay on KPLYR 107.3 FM
- the player does not know jack about the healthbar, he's just slinging tunes
- the healthbar is a big fan of this radio station and tunes in every day and even buys all the exclusive tee shirts to wear

could you imagine if the radio deejay had to drive to your house and play a song for you in person in order for you to hear it? that would be annoying. this makes video games more like radio stations

### basic usage
```lua
local signal = require "signal"

local player = {
    health = 10,
}

-- register a signal on the player
signal.register(player, "health_changed")

player.take_damage = function(self, amount)
    self.health = self.health - amount
    -- emit the signal to tell listeners that health changed
    signal.emit(self, "health_changed", self.health)
end

local health_bar = {}

health_bar.show_health = function(self, new_health)
    print("health: " .. string.rep("#", new_health))
    -- prints "health: ##########"
end

-- connect the signal to the health bar. "show_health" is the connection ID 
-- as well as the method name on health_bar that will be called when the signal
-- is emitted
signal.connect(player, "health_changed", health_bar, "show_health")

-- when the player takes damage, the health bar updates automatically
player:take_damage(1) -- health is now 9
-- prints "health: #########" as the health bar updates


-- important: cleanup when deleting objects
-- this will delete all signals from the object and disconnect all listeners
-- this is important to do when you are done with an object because otherwise 
-- you will have a memory leak and the object can still do things
player:take_damage(1000000)
if player.health <= 0 then
    player = nil
    signal.cleanup(player)
end

-- you can alternatively disconnect a signal without deleting the object
signal.disconnect(player, "health_changed", health_bar, "show_health")

```

### less basic usage
```lua
-- connect has two optional parameters, callback and oneshot
-- signal.connect(emitter, signal_id, listener, connection_id, callback, oneshot)

-- custom callback function instead of method name
signal.connect(player, "health_changed", health_bar, "show_health", 
    function(new_health)
        print("health: " .. new_health)
    end
)

-- signal that only triggers once and then disconnects itself
signal.connect(player, "health_changed", health_bar, "show_health", nil, true)

-- deregister a signal, deletes it and automatically disconnects all listeners
signal.deregister(player, "health_changed")

-- check if a signal exists
local sig = signal.get(player, "health_changed")
if sig then
    -- do something with the signal
end
```

## restrictions
- emitters and listeners must be tables
- signal and connection IDs must be strings or numbers
- each signal must have a unique connection ID to its listener. for example:
  - ✅ you CAN connect signal `"door_opened"` from object `door` to object `lights` with ID `"turn_on"`, and also connect the same signal `"door_opened"` to object `security` with ID `"turn_on"`
  - ❌ you CANNOT connect signal `"door_opened"` from object `door` to object `lights` with ID `"turn_on"` twice
  

## additional notes
- you have to be diligent about cleaning up signals when objects are destroyed. whenever you destroy an object, you should call `signal.cleanup(obj)` unless you want weird things to happen.
- most of the important api calls generate a bit of garbage but it probably won't matter unless you are doing something really insane.
- i dont know if this is thread safe.
