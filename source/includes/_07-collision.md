# Collision detection
_Stop touching meeeeeeee_

We know how to put things on screen, how to make them move under our control but now we need to know when they happen to be one on top of the other.  
To detect this we first need to understand a bit of math on how to know if two rectangles are overlapping, then we will see how to find _global rectangles_ for our PixiJS objects.  
PixiJS has the `Rectangle` class and the `Bounds` class and then PixiJS _DisplayObjects_ have a `getBounds()` method but it returns a `Rectangle` and not a `Bounds`. Please don't think too much about it.

## Pure rectangle math

> Here you have a snippet to check if two rectangles intersect

```ts
// assume `a` and `b` are instances of Rectangle
const rightmostLeft = a.left < b.left ? b.left : a.left;
const leftmostRight = a.right > b.right ? b.right : a.right;

if (leftmostRight <= rightmostLeft)
{
	return false;
}

const bottommostTop = a.top < b.top ? b.top : a.top;
const topmostBottom = a.bottom > b.bottom ? b.bottom : a.bottom;

return topmostBottom > bottommostTop;
```

Ok, first I have to be honest with you, when I said _Rectangles_ I exagerated. We will be working with **Axis-aligned bounding boxes (AABB)** and that means that the sides of our rectangles will always be parallel to one axis and perpendicular to the other. (In layman terms, no rotated rectangles here.)

With that out of the way let's see the naming, a rectangle has `x`, `y`, `width`, and `height`.  
They also have: 
- `top` which is equal to `y`
- `bottom` which is equal to `y + height`
- `left` which is equal to `x`
- `right` which is equal to `x + width`

In our algorithm we now calculate (please read this slowly and thoroughly):
- `rightmostLeft`: Compare the `left` value of our two rectangles and pick the rightmost.
- `leftmostRight`: Compare the `right` value of our two rectangles and pick the leftmost.
- `bottommostTop`: Compare the `top` value of our two rectangles and pick the bottom-most.
- `topmostBottom`: Compare the `bottom` value of our two rectangles and pick the topmost.

If you think about it, I now have a new set of `left`, `right`, `top`, and `bottom` values.  
Here comes the magical part: **if these values make sense, that is to say `left` is to the left of `right` AND `top` is above of `bottom` our initial rectangles are overlapping**.  
As soon as one of the pairs doesn't make sense, we know those rectangles can not be overlapping.

<aside class="info">
Fun fact: When those values make sense, the rectangle that forms is the <b>intersection</b> of your two rectangles! You can use that if you need to separate the rectangles.
</aside>

## DisplayObjects into Rectangles

```ts
function checkCollision(objA: DisplayObject, objB: DisplayObject): boolean {
    const a = objA.getBounds();
    const b = objB.getBounds();

    const rightmostLeft = a.left < b.left ? b.left : a.left;
    const leftmostRight = a.right > b.right ? b.right : a.right;

    if (leftmostRight <= rightmostLeft) {
        return false;
    }

    const bottommostTop = a.top < b.top ? b.top : a.top;
    const topmostBottom = a.bottom > b.bottom ? b.bottom : a.bottom;

    return topmostBottom > bottommostTop;
}
```

Ok, now that we know how to check if two rectangles overlap we just need to make rectangles out of PixiJS DisplayObjects, introducing `getBounds()`  
The _bounds_ of a DisplayObject is an axis-aligned bounding box and in the case of containers, this box contains inside all its children.  
The magic of this is that it works no matter how far apart are the DisplayObjects in the display tree because the method returns the _global_ bounds of any object and that gives us a shared reference system for any pair of objects.

So, the logic is quite simple, we just need to `getBounds()` and then send them into the previous algorithm.

## A note on colliding polygons.

For colliding angled rectangles, polygons, and more information about how they collide you can use the **separating axis theorem (SAT)** that says: _Two convex objects do not overlap if there exists a line (called axis) onto which the two objects' projections do not overlap._  

However, the algorithm is not as trivial as the AABB one, and getting DisplayObjects to become polygonal shapes adds another layer of complexity.

I do have an implementation for this but it's not elegant and I don't feel confident in giving it for the world to use. One day I might write a more robust implementation and make it free to the public.