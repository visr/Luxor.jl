# Polygons and shapes

A polygon is an array of points. The points can be joined with straight lines.

## Regular polygons ("ngons")

You can make regular polygons — from triangles, pentagons, hexagons, septagons, heptagons, octagons, nonagons, decagons, and on-and-on-agons — with `ngon()`.

![n-gons](assets/figures/n-gon.png)

```julia
using Luxor, Colors
Drawing(1200, 1400)

origin()
cols = diverging_palette(60, 120, 20) # hue 60 to hue 120
background(cols[1])
setopacity(0.7)
setline(2)

ngon(0, 0, 500, 8, 0, :clip)

for y in -500:50:500
    for x in -500:50:500
        setcolor(cols[rand(1:20)])
        ngon(x, y, rand(20:25), rand(3:12), 0, :fill)
        setcolor(cols[rand(1:20)])
        ngon(x, y, rand(10:20), rand(3:12), 0, :stroke)
    end
end

finish()
preview()
```

```@docs
ngon
```

## Stars

Use `star()` to make a star. You can draw it immediately, or use the points it can create.

```@example
using Luxor # hide
Drawing(500, 300, "assets/figures/stars.png") # hide
background("white") # hide
origin() # hide
tiles = Tiler(400, 300, 4, 6, margin=5)
for (pos, n) in tiles
    randomhue()
    star(pos, tiles.tilewidth/3, rand(3:8), 0.5, 0, :fill)
end
finish() # hide
nothing # hide
```

![stars](assets/figures/stars.png)

The `ratio` determines the length of the inner radius compared with the outer.

```@example
using Luxor # hide
Drawing(500, 250, "assets/figures/star-ratios.png") # hide
origin() # hide
background("white") # hide
sethue("black") # hide
setline(2) # hide
tiles = Tiler(500, 250, 1, 6, margin=10)
for (pos, n) in tiles
    star(pos, tiles.tilewidth/2, 5, rescale(n, 1, 6, 1, 0), 0, :stroke)
end
finish() # hide
nothing # hide
```

![stars](assets/figures/star-ratios.png)

```@docs
star
```

## Polygons

Use `poly()` to draw lines connecting the points or just fill the area:

```@example
using Luxor # hide
Drawing(600, 250, "assets/figures/simplepoly.png") # hide
background("white") # hide
srand(42) # hide
origin() # hide
sethue("orchid4") # hide
tiles = Tiler(600, 250, 1, 2, margin=20)
tile1, tile2 = collect(tiles)

randompoints = [Point(rand(-100:100), rand(-100:100)) for i in 1:10]

gsave()
translate(tile1[1])
poly(randompoints, :stroke)
grestore()

gsave()
translate(tile2[1])
poly(randompoints, :fill)
grestore()

finish() # hide
nothing # hide
```

![simple poly](assets/figures/simplepoly.png)

```@docs
poly
```

A polygon can contain holes. The `reversepath` keyword changes the direction of the polygon. The following piece of code uses `ngon()` to make and draw two paths, the second forming a hole in the first, to make a hexagonal bolt shape:

```@example
using Luxor # hide
Drawing(400, 250, "assets/figures/holes.png") # hide
background("white") # hide
origin() # hide
setline(5)
sethue("gold")
line(Point(-200, 0), Point(200, 0), :stroke)
sethue("orchid4")
ngon(0, 0, 60, 6, 0, :path)
newsubpath()
ngon(0, 0, 40, 6, 0, :path, reversepath=true)
fillstroke()
finish() # hide
nothing # hide
```
![holes](assets/figures/holes.png)

The `prettypoly()` function can place graphics at each vertex of a polygon. After the polygon action, the supplied `vertexfunction` function is evaluated at each vertex. For example, to mark each vertex of a polygon with a randomly-colored circle:

```@example
using Luxor # hide
Drawing(400, 250, "assets/figures/prettypolybasic.png") # hide
background("white") # hide
origin() # hide
sethue("steelblue4") # hide

apoly = star(O, 70, 7, 0.6, 0, vertices=true)
prettypoly(apoly, :fill, () ->
        begin
            randomhue()
            circle(O, 10, :fill)
        end,
    close=true)
finish() # hide
nothing # hide
```

![prettypoly](assets/figures/prettypolybasic.png)

An optional keyword argument `vertexlabels` lets you pass a function that can
number each vertex. The function can use two arguments, the current vertex number, and the
total number of points in the polygon:

```@example
using Luxor # hide
Drawing(400, 250, "assets/figures/prettypolyvertex.png") # hide
background("white") # hide
origin() # hide
sethue("steelblue4") # hide

apoly = star(O, 80, 5, 0.6, 0, vertices=true)
prettypoly(apoly,
    :stroke,  
    vertexlabels = (n, l) -> (text(string(n, " of ", l), halign=:center)),
    close=true)
finish() # hide
nothing # hide
```

![prettypoly](assets/figures/prettypolyvertex.png)


```@docs
prettypoly
```

Recursive decoration is possible:

```@example
using Luxor # hide
Drawing(400, 260, "assets/figures/prettypolyrecursive.png") # hide
background("white") # hide
srand(42) # hide
origin() # hide
sethue("magenta") # hide
setopacity(0.5) # hide

decorate(pos, p, level) = begin
    if level < 4
        randomhue();
        scale(0.25, 0.25)
        prettypoly(p, :fill, () -> decorate(pos, p, level+1), close=true)
    end
end

apoly = star(O, 100, 7, 0.6, 0, vertices=true)
prettypoly(apoly, :fill, () -> decorate(O, apoly, 1), close=true)
finish() # hide
nothing # hide
```

![prettypoly](assets/figures/prettypolyrecursive.png)

Polygons can be simplified using the Douglas-Peucker algorithm (non-recursive version), via `simplify()`.

```@example
using Luxor # hide
Drawing(600, 500, "assets/figures/simplify.png") # hide
background("white") # hide
origin() # hide
sethue("black") # hide
setline(1) # hide
fontsize(20) # hide
translate(0, -120) # hide
sincurve = [Point(6x, 80sin(x)) for x in -5pi:pi/20:5pi]
prettypoly(collect(sincurve), :stroke,
    () -> begin
            sethue("red")
            circle(O, 3, :fill)
          end)
text(string("number of points: ", length(collect(sincurve))), 0, 100)
translate(0, 200)
simplercurve = simplify(collect(sincurve), 0.5)
prettypoly(simplercurve, :stroke,     
    () -> begin
            sethue("red")
            circle(O, 3, :fill)
          end)
text(string("number of points: ", length(simplercurve)), 0, 100)
finish() # hide
nothing # hide
```
![simplify](assets/figures/simplify.png)

```@docs
simplify
```

The `isinside()` function returns true if a point is inside a polygon.

```@example
using Luxor # hide
Drawing(600, 250, "assets/figures/isinside.png") # hide
background("white") # hide
origin() # hide
setline(0.5)
apolygon = star(O, 100, 5, 0.5, 0, vertices=true)
for n in 1:10000
    apoint = randompoint(Point(-200, -150), Point(200, 150))
    randomhue()
    isinside(apoint, apolygon) ? circle(apoint, 3, :fill) : circle(apoint, .5, :stroke)
end
finish() # hide
nothing # hide
```
![isinside](assets/figures/isinside.png)

```@docs
isinside
```

You can use `randompoint()` and `randompointarray()` to create a random Point or list of Points.

```@example
using Luxor # hide
Drawing(400, 250, "assets/figures/randompoints.png") # hide
background("white") # hide
srand(42) # hide
origin() # hide

pt1 = Point(-100, -100)
pt2 = Point(100, 100)

sethue("gray80")
map(pt -> circle(pt, 6, :fill), (pt1, pt2))
box(pt1, pt2, :stroke)

sethue("red")
circle(randompoint(pt1, pt2), 7, :fill)

sethue("blue")
map(pt -> circle(pt, 2, :fill), randompointarray(pt1, pt2, 100))

finish() # hide
nothing # hide
```

![isinside](assets/figures/randompoints.png)

```@docs
randompoint
randompointarray
```

There are some experimental polygon functions. These don't work well for polygons that aren't simple or where the sides intersect each other, but they sometimes do a reasonable job. For example, here's `polysplit()`:

```@example
using Luxor # hide
Drawing(400, 150, "assets/figures/polysplit.png") # hide
origin() # hide
setopacity(0.7) # hide
srand(42) # hide
sethue("black") # hide
s = squircle(O, 60, 60, vertices=true)
pt1 = Point(0, -120)
pt2 = Point(0, 120)
line(pt1, pt2, :stroke)
poly1, poly2 = polysplit(s, pt1, pt2)
randomhue()
poly(poly1, :fill)
randomhue()
poly(poly2, :fill)
finish() # hide
nothing # hide
```
![polysplit](assets/figures/polysplit.png)

```@docs
polysplit
polysortbydistance
polysortbyangle
polycentroid
```
### Smoothing polygons

Because polygons can have sharp corners, the experimental `polysmooth()` function attempts to insert arcs at the corners and draw the result.

The original polygon is shown in red; the smoothed polygon is shown on top:

```@example
using Luxor # hide
Drawing(600, 250, "assets/figures/polysmooth.png") # hide
origin() # hide
background("white") # hide
setopacity(0.5) # hide
srand(42) # hide
setline(0.7) # hide
tiles = Tiler(600, 250, 1, 5, margin=10)
for (pos, n) in tiles
    p = star(pos, tiles.tilewidth/2 - 2, 5, 0.3, 0, vertices=true)
    setdash("dot")
    sethue("red")
    prettypoly(p, close=true, :stroke)
    setdash("solid")
    sethue("black")
    polysmooth(p, n * 2, :fill)
end

finish() # hide
nothing # hide
```

![polysmooth](assets/figures/polysmooth.png)

The final polygon shows that you can get unexpected results if you attempt to smooth corners by more than the possible amount. The `debug=true` option draws the circles if you want to find out what's going wrong, or if you want to explore the effect in more detail.

```@example
using Luxor # hide
Drawing(600, 250, "assets/figures/polysmooth-pathological.png") # hide
origin() # hide
background("white") # hide
setopacity(0.75) # hide
srand(42) # hide
setline(1) # hide
p = star(O, 60, 5, 0.35, 0, vertices=true)
setdash("dot")
sethue("red")
prettypoly(p, close=true, :stroke)
setdash("solid")
sethue("black")
polysmooth(p, 40, :fill, debug=true)
finish() # hide
nothing # hide
```

![polysmooth](assets/figures/polysmooth-pathological.png)

```@docs
polysmooth
```

### Offsetting polygons

The experimental `offsetpoly()` function constructs an outline polygon outside or inside an existing polygon. In the following example, the dotted red polygon is the original, the black polygons have positive offsets and surround the original, the cyan polygons have negative offsets and run inside the original. Use `poly()` to draw the result returned by `offsetpoly()`.

```@example
using Luxor # hide
Drawing(600, 250, "assets/figures/polyoffset-simple.png") # hide
origin() # hide
background("white") # hide
srand(42) # hide
setline(1.5) # hide

p = star(O, 45, 5, 0.5, 0, vertices=true)
sethue("red")
setdash("dot")
poly(p, :stroke, close=true)
setdash("solid")
sethue("black")

poly(offsetpoly(p, 20), :stroke, close=true)
poly(offsetpoly(p, 25), :stroke, close=true)
poly(offsetpoly(p, 30), :stroke, close=true)
poly(offsetpoly(p, 35), :stroke, close=true)

sethue("darkcyan")

poly(offsetpoly(p, -10), :stroke, close=true)
poly(offsetpoly(p, -15), :stroke, close=true)
poly(offsetpoly(p, -20), :stroke, close=true)
finish() # hide
nothing # hide
```

![offset poly](assets/figures/polyoffset-simple.png)

The function is intended for simple cases, and it can go wrong if pushed too far. Sometimes the offset distances can be larger than the polygon segments, and things will start to go wrong. In this example, the offset goes so far negative that the polygon overshoots the origin, becomes inverted and starts getting larger again.

![offset poly problem](assets/figures/polygon-offset.gif)

```@docs
offsetpoly
```

### Perimeter utilities

`polyperimeter` calculates the distance of a polygon.

```@example
using Luxor # hide
Drawing(600, 250, "assets/figures/polyperimeter.png") # hide
origin() # hide
background("white") # hide
srand(42) # hide
setline(1.5) # hide
sethue("black") # hide
fontsize(20) # hide
p = box(O, 50, 50, vertices=true)
poly(p, :stroke)
text(string(round(polyperimeter(p, closed=false))), O.x, O.y + 60)

translate(200, 0)

poly(p, :stroke, close=true)
text(string(round(polyperimeter(p, closed=true))), O.x, O.y + 60)

finish() # hide
nothing # hide
```

![polyperimeter](assets/figures/polyperimeter.png)

`polyportion()` and `polyremainder()` return part of a polygon depending on the fraction you supply. For example, `polyportion(p, 0.5)` returns the first half of polygon `p`, `polyremainder(p, .75)` returns the last quarter of it.

```@example
using Luxor # hide
Drawing(600, 250, "assets/figures/polyportion.png") # hide
origin() # hide
background("white") # hide
srand(42) # hide
setline(1.5) # hide
sethue("black") # hide
fontsize(20) # hide

p = ngon(O, 100, 7, 0, vertices=true)
poly(p, :stroke, close=true)
setopacity(0.75)

setline(20)
sethue("red")
poly(polyportion(p, 0.25), :stroke)

setline(10)
sethue("green")
poly(polyportion(p, 0.5), :stroke)

setline(5)
sethue("blue")
poly(polyportion(p, 0.75), :stroke)

setline(1)
circle(polyremainder(p, 0.75)[1], 5, :stroke)

finish() # hide
nothing # hide
```

![polyportion](assets/figures/polyportion.png)

`polydistances` returns an array of the accumulated side lengths of a polygon.

    julia> p = ngon(O, 100, 7, 0, vertices=true);
    julia> polydistances(p)
    8-element Array{Real,1}:
       0.0000
      86.7767
     173.553
     260.33  
     347.107
     433.884
     520.66  
     607.437

`nearestindex` returns the index of the nearest index value, an array of distances made by polydistances, to the value, and the excess value.

```@docs
polyperimeter
polyportion
polydistances
nearestindex
```

### Fitting splines

The experimental `polyfit()` function constructs a B-spline that follows the points approximately.

```@example
using Luxor # hide
Drawing(600, 250, "assets/figures/polyfit.png") # hide
origin() # hide
background("white") # hide
srand(42) # hide

pts = [Point(x, rand(-100:100)) for x in -280:30:280]
setopacity(0.7)
sethue("red")
prettypoly(pts, :none, () -> circle(O, 5, :fill))
sethue("darkmagenta")
poly(polyfit(pts, 200), :stroke)

finish() # hide
nothing # hide
```

![offset poly](assets/figures/polyfit.png)

```@docs
polyfit
```

## Converting paths to polygons

You can convert the current path to an array of polygons, using `pathtopoly()`.

In the next example, the path consists of a number of paths, some of which are subpaths, which form the holes.

```@example
using Luxor # hide
Drawing(800, 300, "assets/figures/path-to-poly.png") # hide
background("white") # hide
origin() # hide
fontsize(60) # hide
translate(-300, -50) # hide
textpath("get polygons from paths")
plist = pathtopoly()
setline(0.5) # hide
for (n, pgon) in enumerate(plist)
    randomhue()
    prettypoly(pgon, :stroke, close=true)
    gsave()
    translate(0, 100)
    poly(polysortbyangle(pgon, polycentroid(pgon)), :stroke, close=true)
    grestore()
end
finish() # hide
nothing # hide
```

![path to polygon](assets/figures/path-to-poly.png)

The `pathtopoly()` function calls `getpathflat()` to convert the current path to an array of polygons, with each curved section flattened to line segments.

The `getpath()` function gets the current path as an array of elements, lines, and unflattened curves.

```@docs
pathtopoly
getpath
getpathflat
```
