---
layout: page
title: Event Introduction
bigimg: /img/header_twilight.jpg
---



There are many situations where a mod author may want to change vanilla behavior, or have their code execute on a certain trigger. Forge's solution to this is their event system. Forge's event system is based on moderatly advanced concepts which can make it feel daunting to new devs. This tutorial should help you get a grasp of events, and give you a glimpse into how it works behind the scenes. Getting comfortable with events truly opens up a whole new world of posibilities for mod authors and can help set your mod apart from the rest. 

For this tutorial I will be showing how to have code trigger when an item is picked up although events come in all shapes and sizes, and the things demonstrated in this tutorial can be applied to them as well. There is currently no official list of all events, however [VicNightfall](https://twitter.com/VicNightfall) made [this](https://dl.dropboxusercontent.com/s/h777x7ugherqs0w/forgeevents.html) list a while back. That list is a bit outdated but it is the most complete list I know. 

The first thing to do when using events is finding where you want to put them. Most devs will create a new class somewhere in their mod called `ModNameEventHandler` or something similar and dump all of their events in there. The name isn't very important, however it's important to avoid the name `EventHandler` as it will conflict with Forge's annotation class of the same name. Once you have your class, you need to define a new event method. An event method is similar to a normal method however it does have a few conditions. Here is a list of those conditions. 

- Any name can be used for the method.
- Must return void.
- Must be public.
- Must have only one parameter, which is a type that extends Forge's event class!
- Must have the `@SubscribeEvent` annotation above it.
- Method must **NOT** be static.

So, in the case of this tutorial I am using `ItemPickupEvent` so my event method will look like this. 

```java
    @SubscribeEvent
    public void onItemPickedUp(ItemPickupEvent event) {
        
    }
```

Now that the event has been defined, we need to register the class which holds it with forge's event bus. This can be done by calling `MinecraftForge.EVENT_BUS.register(new YourHandlerHere());`. This can technicaly be done at any stage of mod loading, however most people do it during the preInit stage, which I would recommend. Now that you have your event registered, the method you created before should be called every time the event is fired. In my case it is whenever the player picks up an item. 

So, what can you do with events? One of the main use cases is reacting to the event. For example, if you want to trigger an achievement when your item is picked up, you could do that from within your event method. The event parameter is provided by Forge's event system when the method is called and contains all of the context for the situation. In the case of `ItemPickupEvent` the player who picked up the item and the item being picked up are both public fields, however you usually need to use a getter/setter method to interact with the context. In this example I am giving the player the "when pigs fly" achievement when the pick up an Elytra. 

```java
    @SubscribeEvent
    public void onItemPickedUp(ItemPickupEvent event) {
        
        if (event.player != null && event.pickedUp.getEntityItem().getItem() instanceof ItemElytra)
           event.player.addStat(AchievementList.FLY_PIG);
    }
```

You can also use events to change the context of the situation. This is a bit different with each event, but you can usually get away with using a setter field from the event passed. In this case I have access to the item being picked up, and I can change it to something else. For example, if the player would pick up a block of dirt, I can change that dirt into a stack of diamonds instead. 

```java
    @SubscribeEvent
    public void onItemPickedUp(ItemPickupEvent event) {
        
        if (event.player != null && Block.getBlockFromItem(event.pickedUp.getEntityItem().getItem()) instanceof BlockDirt)
            event.pickedUp.setEntityItemStack(new ItemStack(Items.DIAMOND, event.pickedUp.getEntityItem().stackSize));
    }
```

You can also completely prevent the event from happening if you want. This is done by using `setCanceled(true)` on the event parameter. Be careful when using this, as not all events can be canceled and if you try to cancel an event that is not cancelable forge will intentionally crash the game on you. In this example I prevent the player frome ver picking up a diamond. 

```java
    @SubscribeEvent
    public void onItemPickedUp(ItemPickupEvent event) {
        
        if (event.player != null && event.pickedUp.getEntityItem().getItem() == Items.DIAMOND)
            event.setCanceled(true);
    }
```

You may also notice that some events use results rather than being canceled. Results are not very standardised so nearly every event that uses them will use them differently, most don't use them at all. The purpose of results is to give more control over an event to the dev using it. There are three result types, the first is `DEFAULT`, this usually means the event will progress as normal. The second is `ALLOW`, which typically allows the event to apply non-vanilla logic to the situation. Lastly there is `DENY` which is usually similar to cancelling the event.

In some situations you may also need to change the priority of your event. This can be done by adding parameters to the annotation when you add it to the method. The `priority` parameter is used to determin the order in which events are executed. By default your event is set to `NORMAL` priority. There is also `HIGHESt', `HIGH`, `LOW` and `LOWEST`. An event with a lower priority is called last, while an event with higher priority is called first. Here is an example of what this might look like. 

```
    @SubscribeEvent(priority = EventPriority.HIGHEST)
    public void onItemPickedUp(ItemPickupEvent event) {
        
        if (event.player != null && event.pickedUp.getEntityItem().getItem() == Items.DIAMOND)
            event.setCanceled(true);
    }
```

There is another annotation parameter for recieving an event that has been canceled. This parameter is `receiveCanceled`. Normally if an event has been canceled, the other listenders for that event will not recieve the event. If you set this to true, your event will be called regardless. This is mostly useful if you want to uncancel an event. To give an example, lets say mod A cancels the pickup event, if that happens mod B's event method will not be fired unless they have `receiveCanceled` set to true. Here is an example of what this would look like. 

```java
    @SubscribeEvent(receiveCanceled = true)
    public void onItemPickedUp(ItemPickupEvent event) {
        
        if (event.player != null && event.pickedUp.getEntityItem().getItem() == Items.DIAMOND)
            event.setCanceled(false);
    }
```

This covers all of the basics when it comes to events, but some of you may be curious how this all works underneath the surface. Annotations such as `SubscribeEvent` are a feature of Java which allows the author to add metadata to their source code. Unlike java docs or comments, they are compiled with your source and can be read. When you call `MinecraftForge.EVENT_BUS.register` Forge reads the meta data for the instance you provide, and uses the `SubscribeEvent` annotations as a way to find these methods. Once a target method is found, a reference to it can be stored, which is later used by forge when one of it's hooks are called.
