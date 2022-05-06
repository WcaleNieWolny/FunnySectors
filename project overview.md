# Funny Sectors development plan

> Note: Please be aware - this is all hypothetical, I am not sure if any of it is possible.
> There will be a lot of my opinion here but I would love to discuss

## Project language

I think we should use Kotlin. We are already using kotlin in [FunnySkAddon](https://github.com/FunnyGuilds/FunnySkAddon) and [FunnyDrama](https://github.com/FunnyGuilds/FunnyDrama)
Kotlin is modern and it is great at removing boilerplate code. In sectors there will be a lot of try and catch blocks and kotlin does not require that.
Another reason why using kotlin is a great idea is build-in [loombok](https://projectlombok.org/)
We can't do getters and setters for a packets manually

## Build System

Gradle - this is obviously a better build system for this project.
FunnyGuilds migrated to it a while ago and this project probably will be using a lot of modules

## Version support

I think we should support 1.17 (latest), 1.18 (all) and any incoming versions
Supporting 1.8 is a bad idea. This is very [old](https://howoldisminecraft188.today/) version of minecraft
If we support it we will be supporting legacy code that never should be used again
Now... I know that this is controversial, but I am not trying to overcome any issues caused by legacy code
however I think it is important to build this plug-in in a way that would allow 1.8 to work by having a separate module

## Plugin platform

Honestly? I would just support [velocity](https://velocitypowered.com/) and have a separate server hosted on velocity to handle communication
We also could use Redis pub/sub to send data however this should not be an only way to run the plugin
Perhaps we would also support [BungeeCord](https://www.spigotmc.org/wiki/bungeecord/) but this is up to discussions
Please be aware that I am against supporting it

## Code structure

Good structuring of the code is very important
I think the code should be versatile so adding support for new versions is easy

Here is my idea how to do it:

```
shared - contains all of the shared logic of the plugin
velocity - contains all of the velocity implementation
bungee (see plugin platform)
api - developer API used to serialize data between servers
nms - all of the classes for version specific implementation
1_17 - implementation of "nms" for 1.17
1_18 - implementation of "nms" for 1.18
```

## Plugin modes

The plugin shoudl have 2 modes

- static
- dynamic

Static mode would work as any other sectors plugin - it would divide map into X amount of server
Each of the servers would have a border that would send player to another server (sector)

Dynamic mode is a little more difficult to explain
It would require a server to be running in cloud (any provider)
When a sector detect an area that is taking a lot of its resources (big players event etc) it would attempt to create another sector inside it
this new sector would be assigned a new ID and from now it would be responsible for these chunks and arena around it (lets say 5 chunks around)
Then the controller server (proxy) would request a temporary server from a cloud provider (preferably docker container)
After the new sector starts up it will open an [tcp](https://en.wikipedia.org/wiki/Transmission_Control_Protocol) server that would receive direct connection from parent server and notify proxy to close all players connections to parent server

After that there will be to options for data transfer:
- soft
- hard

If sector [tps](https://minecraft.fandom.com/wiki/Tick) is below threshold it would use hard mode else it would use soft mode
Soft mode would split the laggy area into small parts and schedule a few sync tasks to obtain data from those chunks
After that it would send it in parts to the tcp server on child server
Hard mode just schedules one synchronized data transfer
If hard mode is used the proxy will be notified and it will close all of the active connections to that sector without closing connections to proxy
All of the players on that sector will be in a limbo state
They will be connected to a simple (completely custom) server that would just accept their movement and show movement of other players
This limbo server would be optymalized to use very small amount of resources and it would be always running
Perhaps it would start when a subsector is requested - this is up to disscusions
After the data transfer is finished the TCP server on subsector will be closed and players will be reconnected to sectors
If someone was not near the bussy area he will stay on parent sector. If not he will be sent to new sector
The new sector has to have a boundry. It will be set according to amount of chunks for subsector and its position
I have left pictures below

>Note: Red is the laggy area, green is the normal area, orange is the old sector, purple is the new sector

![](https://raw.githubusercontent.com/WcaleNieWolny/FunnySectors/main/img/img1.png)

In this situation the bussy area is near two borders
We do not want to have that. It woudl do something like this:

![](https://raw.githubusercontent.com/WcaleNieWolny/FunnySectors/main/img/img2.png)

Now... let's see another example

![](https://raw.githubusercontent.com/WcaleNieWolny/FunnySectors/main/img/img3.png)

It creates a problem - we can't just expand the sector to nearest border becouse we need to save cloud resources
In that case we need to do something like this:

![](https://raw.githubusercontent.com/WcaleNieWolny/FunnySectors/main/img/img4.png)

We expanded a little the area, but it is still no touching any border
In that case we would have to unload the chunks from parent sector and let the child server handle it
But as you may figure out it creates another issue. We have a big hole and no way to show it on parent server
It is very clearly visible but there is no data about this chunk
We could just use the old data but memory is valuable
That is why we do not onload 2 chunks from the border
Furthermore we run an algorithm to create very simple terrain that would be maximally 16 blocks tall
That would create a simple terrain that would not look out-of-place without taking a lot of resources
We also could indicate that the border is nearby using particles

There also will be a background task for checking if you coudl merge two servers together
If child server is no longer required the merging process will start
All the unloaded chunks will be sent to parent server
It will be happening in the course of few (around 20?) ticks
In that time loading any new chunk on subsector will not be possible
After that all of the players will be sent to libo server where all of their actions will be saved
Meanwhile the child server will be dumping all of its data to the parents server
When that is completed proxy will move players from limbo to merged sector and child server could get destroyed

## Entity ID

As you might figure out moving players without directly resending them between them is problematic
There will be a lot of ID conflicts
To solve it all the servers will inject their sector ID after the normal entity ID
That means that sector ID must be unique
In dynamic mode after a server is merged the parent server will keep using child entity ID

## Conclusion

This is a very difficult project that require a lot of effort to get done.
But I hope that with [Bookkity](https://discord.gg/CYvyq3u) community we could make it.
I believe that with the brilliant people of this discord it's possible
Let's create a sectors plugin that the Polish community desperly needs
Also please do not judge me for english - I did not use autocorrect when writing it
