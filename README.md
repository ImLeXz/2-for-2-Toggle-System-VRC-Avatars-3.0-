# 2 for 2 Toggle System VRC Avatars 3.0

### This largely out of date and was done before avatars 3.0 allowed for bools or the increased parameter data

### Toggle system for VRChat avatars 3.0, making use of 2 networked integers and animator bools to make up to 255 toggles from just 2 vrc parameters

### How It Works

This toggle system makes use of the **VRC Avatar Parameter Driver** component to set bool parameters on the animator to true or false. This component sets the bool state for all animators too, so if a bool is set to false from a state on one animator, the bool of the same name on another parameter will also change to false. This is very useful as it allows us to store our VRC Int Parameter value as a bool parameter on our animators.

### The Problems And How They Were Solved

Since parameters stored on our animators aren't sync'd over the network like the VRC parameters are, late joiners will not know when a toggle has been activated, causing desync issues. To get around this, I built a syncing system that sets a VRC parameter to specific values that correspond to the animator bool states on a set interval. This allows the toggles to sync up for late joiners, but also gives the user the option to add another parameter which they can toggle that determines whether the toggle should sync or not.

Some issues with the system still remain however. 

First being that you can't click a toggle whilst it's syncing so sometimes it may feel like the system isn't working but it may just be because it's syncing. This could be improved upon in a few ways, either by making all toggles into one layer so it's less time syncing but this could cause stability issues with desync, or not syncing whilst the menu is open. Not syncing whilst the menu is open would probably be the best way to mitigate this issue but currently there isn't a good way to detect whether the menu is fully closed (I believe), there is a way to set a parameter when a user is inside of a sub menu, but that parameter doesn't reset when the user closes the action menu entirely, only when they back out of the sub menu so it isn't the best approach.

One other issue is that syncing can take a little while for late joiners, this is because of the default 10 second interval so the animations aren't constantly looping and the issue with pressing toggles whilst syncing (mentioned above) isn't as obvious. This can be altered by changing the syncing interval but then the first issue becomes worse, so mitigating first issue is more important. If VRC adds an internal bool parameter that checks if the player has the action menu open or not then this would help a lot.

## Tutorial

### Introduction And What's Included In The Package

The package contains **VRC Avatars SDK 3.0 (Newest Version As 01/09/2020)** and an example scene with an avatar setup with 3 different types of toggles, which are: Group Toggles (only one toggle in the group can be active at once), Individual Toggles (all toggles of this type can be active at once), On To Off toggles (Like individual toggles, however their default state is on).

Depending on what you want your toggle animation to do, they may need to either go on the FX layer, or the action layer, but this is up to you to decide and the official guides on the ask.vrchat forums offer a good explanation into what each layer should be used for.

In my example I made it so that all the syncing and resets are done on the action layer. This is because when I had them on the FX layer I was running into issues where the syncing would repeat twice, causing some issues with other functionality I was working on. I came to the conclusion that this was due to things being duplicated so that they can work in the mirror for the FX layer, however since stuff on the Action layer doesn't run in the mirror reflection, moving the syncs to that layer seemed to have fixed the issue.

Each toggle that is on the avatar is done using a **Button** control in the **VRC Expressions menu** so that the value is set to a specific number then back to 0 when released.
For the two VRC Int Parameters that are used for toggles, there is **m_ToggleNum** which is used for the first button press that sets the toggle to its toggled state, then there is **m_ResetNum** that is used for the second button press which sets the toggle state back to default. Then each toggle needs to have a **bool parameter** on the animator which will store the toggle states as the **m_ToggleNum** gets reset to 0. I also like to add which **m_ToggleNum** value the **bool parameter** corresponds to at the end of the bool name, for example: **m_ToggleNum = 1** turns Green Cube on, so Green Cube bool is named **IsGreenCubeOn (1)**.

Pretty much all layers in my animators start from the entry state and transition straight to an **Idle** state so that I can run a condition from that to another state.

### Group Toggles

The group toggle included in the example features a 3-way object toggle group where only one of the objects can be enabled at once.
Each one of these toggles has a separate layer in the animator as there were issues with running them from Any State on a single layer, and work as follows:

***Here I will cover how the first toggle for the group works. (Toggles Green Cube On / Off, Toggles Green Sphere Off, and Toggles Green Capsule Off)***

**(Idle -> Wait)** is the first transition, that has a condition which checks whether the **m_ToggleNum** is equal to 1.

**(Wait -> Turn On)** Condition checks **m_ToggleNum** is back to 0 to stop animation from going straight to exit.

**(Turn On)** This animation state is where you would put your toggle animation (The thing you want your toggle to do), and is also where the first **VRC Avatar Parameter Driver** component is placed which sets the **bool parameters: IsGreenCubeOn (1) to true, IsGreenSphereOn (2) to false, IsGreenCapsuleOn (3) to false** so that the other objects in the group are disabled.

**(Turn On -> Turn Off)** There is two transitions between these two states, the first one checking whether the **m_ResetNum** is equal to 1 (value corresponds to **m_ToggleNum**), Second checking if **IsGreenCubeOn (1)** is false. This means it will transition to the off state if **m_ResetNum** is 1, **OR IsGreenCubeOn (1)** is false.

**(Turn Off)** This state is where the second **VRC Avatar Parameter Driver** component is. This one sets **IsGreenCubeOn (1)** parameter back to false. This state also has the **- Frame Delay** animation which is used to add a short delay to make sure the parameter is set remotely before transitioning. 

**(Turn Off - Exit)** Transition checks whether **m_ToggleNum** is 0 to stop the layer repeating, and **IsGreenCubeOn (1)** is false to make sure the parameter as been set by the component.


### Individual Toggles
Toggles Blue Cube, Blue Sphere and Blue Capsule from OFF to ON. All these can be turned on at the same time.

***Here I will cover how this type of toggle works.***

*This works exactly the same way as the group toggles, however the first **VRC Avatar Parameter Driver** component only sets the bool parameter to true, and none of the others to false.*

### On To Off Toggles
Toggles Red Cube, Red Sphere, and Red Capsule from ON to OFF. All these can be turned off at the same time.

***Here I will cover how this type of toggle works.***

*This type of toggle works fairly similarly to the individual toggles, however some of the parameters are reversed*
Unlike all the other bools used, these bools all are default on so things need to be inverted.
Instead of the first **VRC Avatar Parameter Driver** component setting the parameter to true, it sets the parameter to false.
The transition between the two states with the **VRC Avatar Parameter Driver** components on then checks if the bool is true, instead of checking if it's false
The second **VRC Avatar Parameter Driver** component then sets the parameter back to true.
Then the final transition into the exit state checks if the parameter is true, instead of checking if it's false.


### Resets Layer

The Resets layer is located on the **Action Layer** animator and is used for determining whether the user is turning the toggle on or off.

**(Idle -> IsLocal)** Checks whether the internal VRC parameter **IsLocal** is true

***This is needed so that the resets don't play twice or out of sync remotely***

**(IsLocal -> IsSyncing FALSE)** Checks whether animator bool **IsSyncing** is false with a short exit time for stability.

**(IsSyncing FALSE -> IsLocal)** Checks whether animator bool **IsSyncing** is true.

***These checks are needed so that the sync system doesn't trigger the resets, and only the user clicking the button triggers them***

From **IsSyncing FALSE**, there is then a bunch of transitions that go to all the different resets for the toggles, each of these transitions checks whether the **m_ToggleNum** is a specific value, and corresponding bool is true, then transitions to the reset for that specific toggle. The only difference is for the *On To Off* toggles, where those reset transitions check if the bool is false.

Every reset then has a **VRC Avatar Parameter Driver** component that sets **m_ResetNum** to the corresponding value to the **m_ToggleNum**. Each reset state also has the **- Frame Delay** animation too, to add a short delay for extra stability, then transitions with an exit time of 1 and no conditions to a Wait state.

**(Wait)** Wait state uses **VRC Avatar Parameter Driver** to set **m_ResetNum** back to 0, als has **- Frame Delay** animation for the delay.

**(Wait -> Exit)** Exit time of 1 to make sure **m_ResetNum** is set remotely before transitioning to exit.


### Sync Layers

Pretty much all the sync layers work in the same away and are located on the **Action Layer** animator, and is used to sync the toggles for late joiners so that they are in the correct state.

**(Idle -> IsLocal)** Checks whether internal VRC Parameter **IsLocal** is true.

**(IsLocal -> CheckSync)** Checks whether bool **IsSyncing** is false, to make sure two toggles aren't syncing at the same time, transitions back to **IsLocal** if the bool changes back to true

**(CheckSync -> Set IsSyncing TRUE)** Checks Whether bool for the toggle that is being sync'd is true

**(Set IsSyncing TRUE)** Uses **VRC Avatar Parameter Driver** component to set bool **IsSyncing** to true

**(Set Is Syncing TRUE -> Sync)** Checks if bool **IsSyncing** is true

**(Sync)** Uses **VRC Avatar Parameter Driver** component to set **m_ToggleNum** to the value corresponding to the toggle, also uses **- Frame Delay** animation.

**(Sync -> Reset ToggleNum)** Exit time of 1.

**(Reset ToggleNum)** Uses **VRC Avatar Parameter Driver** component to set **m_ToggleNum** back to 0, also uses **- Frame Delay** animation.

**(Reset ToggleNum -> Set IsSyncing FALSE)** Checks if **m_ToggleNum** is 0, has Exit Time of 1.

**(Set IsSyncing FALSE)** Uses **VRC Avatar Parameter Driver** component to set bool **IsSyncing** to false

**(Set IsSyncing FALSE -> Sync Interval)** Checks if bool **IsSyncing** is false

**(Sync Interval -> Exit)** Exit Time of 10, syncing the toggle every 10 seconds
