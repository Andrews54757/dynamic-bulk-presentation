Slide 1:

Hello llama engineers,

Welcome back to another lecture on storage technology.

Slide 2:

Today, we'll dive into encoded dynamic bulk systems—what they are, how they function, and how to enhance them using techniques I’ve developed.

In essence, a dynamic bulk storage system automatically adjusts capacity based on the quantity of items stored. Unlike static bulk systems, which have a fixed capacity that must be manually set for each item type, dynamic bulk systems determine the way storage capacity is organized on their own, in order to maximize efficiency without human intervention.


Slide 3:

To create a dynamic bulk system, the first step is to divide the storage capacity into fixed-size slices. Typically, a slice includes a couple of double chests, offering a capacity of about half a million to a million items. It’s important to choose slice sizes wisely, because it determines the smallest amount of storage that can be allocated to any item type. It wouldn’t make much sense to have large slices if you only have few amount of most item types.

Slide 4:

Let’s break down the basic logic of a dynamic bulk system with a simple example. Suppose we want to store eight chests of redstone in a dynamic bulk storage system that is currently empty. The first task of the system is to check if there’s an existing slice for redstone. If there is, the items can be stored there. However, in this case, there isn’t a slice allocated yet, so the system will need to allocate a new slice for redstone.

Slide 5:

We can store five chests of redstone in that newly allocated slice before it becomes full. However, with three more chests to store, another slice must be allocated to accommodate the remaining items.

Slide 6:

In the end, we have one fully filled slice of redstone and another slice with some leftover storage space. This non-full slice is known as the partial slice.

Slide 7:

Tracking the partial slice is crucial because it’s best to empty it first when retrieving items. This strategy prevents cumulative fragmentation, which can lead to wasted storage space over time.

Slide 8:

From our example we learned that we need to keep track of what slices are assigned to each item type. This information must be stored in a way that allows for quick access. Remember, the system must query the system for slices allocated to an item type before doing anything else, and any delay in this step will increase the wait time for the requested items.

Another consideration is size. Ideally, the bulk storage should be expandable without becoming unbearably big. A small system is also easier to build.

To understand the challenges of developing dynamic bulk systems, we need to take a look at their history.


Slide 9:

In the beginning, there was PallaPalla’s designs, and for a time, it was good. But PallaPalla’s implementation of his “self-organizing bulk storage” system, fell victim to issues with stability and scalability.

These problems stemmed from the way data storage was integrated into each slice. Palla used a re-mappable binary decoder that relied on toggle states to match incoming requests with the assigned slices.

This approach required every piston in each slice to be activated for an input bit, causing lag and increasing the risk of data corruption. Additionally, the slices themselves were massive, making it difficult to scale the system.


Slide 10:

Problems with Palla’s design led to the development of the external logic dynamic bulk concept. With this approach, bulk slices could be simplified into static devices, each with a fixed address.

The idea was that by moving slice allocation information to a separate, specialized module, the system could become simpler, smaller, and easier to implement compared to dynamic bulk systems with internal logic.

Slide 11:

Plans for the system were as follows. Each bulk storage slice is represented by an item called the slice code item. An encoder reads the slice code item and unlocks the corresponding slice.

These slice code items are stored in a shulker box dedicated to each item type, keeping track of the slices allocated for that type.

To make this work, each box associated with an item type needs to be stored in a way that allows it to be quickly retrieved based on the requested item code.

This means we need a separate callable storage system just to manage our bulk slices. To store and retrieve one specific box out of a thousand for different item types, we’ll need at least a thousand addresses in that callable item storage. So, how do we build a device that can store and retrieve one specific box out of a thousand?

Slide 12:

One way to achieve this is by using an item RAM. An item RAM is a device that can store and retrieve items from a specific address in constant time. It’s like a computer’s RAM, but for items. 

The fast constant time retrieval is achieved by dedicating a seperate block to store items for each address, typically a dropper, and using a redstone logic to activate the block at the desired address. This method allows for the fastest possible retrieval time, but it requires a lot of space and redstone to build.

Slide 13:

The large size of the item RAM is a significant drawback, and so people have been looking for ways to reduce the size of the device by increasing density. Latest advancements have doubled the density, halving the size of the device, without sacrificing retrieval speed. However, the device is still prohibitively large for most applications, and the search for a more compact solution continues.

Slide 14:

On the other end of the spectrum, we have the item disk drive. The item disk drive is a device that can store and retrieve items from a specific address in linear time. It's much compact than the item RAM, but it's slower. It stores multiple addresses in a double chest, and the retrieval time is proportional to the number of addresses stored in the chest because it has to cycle through the chest to find the desired item.

Slide 15:

The speed at which the chest can be cycled is typically limited by hopperspeed, meaning that the maximum speed is 8 game ticks per item. A box at the 32nd slot will take a minimum of 248 game ticks to retrieve, which is about 12.4 seconds. 12.4 seconds is a long time to wait for an item, so the item disk drive is not suitable for systems that require fast retrieval.

Slide 16:

To recap, item RAMs are too big, and disk drives are too slow. The ideal solution would be a device that combines the best of both worlds to have both fast retrieval time and a compact size. Such a device would revolutionize the way we build dynamic bulk storage systems, making them more efficient and easier to implement.

How can we achieve this? To answer that question, we need to seek inspiration from the world of computer science to develop a new  approach to dynamic bulk storage systems.

Slide 17:

Lets make some observations. Recall that it is important to retrieve a specific box out of a thousand quickly because any delay in this step will increase the wait time for the requested items to arrive. But does the system need to know all the slices allocated to an item type at once?

No, it doesn't. The system only needs to know the partial slice first, as it is the slice that will be emptied or filled first. Filling and emptying takes a while anyway, so the system can take its time to find the other slices allocated to the item type. We only need to be quick with queries for the partial slice and the rest can be done in the background.

Slide 18:

This observation leads us to the concept of lazy loading. Lazy loading is a design pattern commonly used in computer programming to defer loading an object until the point at which it is needed. In our case, we can apply lazy loading to the storage of slice allocation information.

Instead of storing all the slices allocated to an item type in a single storage device, we can store only the partial slice in a separate storage device. When the system needs to access the other slices, it can do so in the background without affecting the retrieval speed of the partial slice.

Slide 19:

A data structure that best fits this concept is the linked list. A linked list is a data structure that consists of a sequence of elements, where each element points to the next element in the sequence. This structure allows for efficient insertion and deletion of elements at the start of the list, at the cost of slower access time to elements in the middle of the list.

With a linked list, instead of storing slice code items for all the slices allocated to an item type in a single storage device, we can store only the partial slice code item in the device. When the system needs to access the other slices, it can do so by following the links in the list, which are stored in the slices themselves.

Slide 20:

Only needing to store one slice code item per item type in a storage device makes the device much smaller and easier to build. These items are not boxes, so they can be stored in boxes themselves. This means we have multiple addresses in a single box, which is a much more compact solution. The boxes can then be stored in an item RAM or an item disk drive, depending on the retrieval speed required.

Slide 21:

But wait! We still need to cycle through items in the box to find the desired item, which is slow. How can we speed up the retrieval process?

The answer lies in using hoppercarts. Hoppercarts have an 8 times faster transfer rate than hoppers, which means that the maximum speed of cycling can be increased to 1 game tick per item. At this speed, the 20th slot will take only 20 game ticks to retrieve, which is about 1 second.

That we need to cycle through boxes and not chests is another significant advantage. Unlike chests, a box can be easily transported using instant dropper lines to the hoppercart. Thus, one cart cycler can be built for multiple boxes, making the system even more compact and efficient.

Slide 22:

The result of combining a hoppercart cycler with a small item RAM is a device that has both a compact size and fast retrieval times for all adressess. This non-box-item RAM can retrieve one specific item out of a thousand in merely 55gt, which is about 2.75 seconds. All while being comparable in size to an item disk drive.

Slide 23:

So what does a linked list dynamic bulk storage system look like in practice? Here is the logic it follows to store items:

First, the system obtains a slice code item at a slot corresponding to the item code using the 1000 non-box-item RAM. Then, it inserts boxes into the corresponding slice. If the slice is full, the system pulls a new slice code item from storage to allocate a new empty slice. The now-full slice code item is put into the newly allocated slice, and the rest of the boxes are put into the newly allocated slice. This process is repeated as needed. Finally, the remaining slice code item is stored into the 1000 non-box-item RAM at the slot corresponding to the item code to finish insertion.

Slide 24:

The logic to retrieve items is similarly simple: 

First, the system obtains a slice code item at a slot corresponding to the item code using the 1000 non-box-item RAM. Then, it retrieves items from the corresponding slice, testing if they are boxes. If an item is not a box, it is held for later, as it is the slice code item for the next slice. If the slice is empty and a code item for the next slice is found, the system switches to the next slice and puts the now-empty slice code item back into storage. This process is repeated as needed. The slice code item for the next slice is put back into the slice if found, and the remaining slice code item is stored into the 1000 non-box-item RAM at the slot corresponding to the item code to finish retrieval.

Slide 25:

In conclusion, the linked list dynamic bulk storage system is a revolutionary approach to bulk storage that combines the best of both worlds: fast retrieval times and a compact size. By using lazy loading and linked lists, we can build a system that is efficient, easy to implement, and scalable. 

All that is left now is to build it. I have posted the schematic of the 1000 address non-box item RAM in the description to help you get started. I unfortunately have no time to implement this myself, but I hope you will take on the challenge and be the first to build a practical dynamic bulk storage system.

So Good luck, and happy engineering!