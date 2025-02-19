# tiny-brot

A fractal demo.

<img src="tiny-brot.gif" title="Playing the game">

## Playing

tiny-brot draws the mandelbrot set, press <kbd>enter</kbd> to zoom.

Start by running [./tiny-brot.hs](tiny-brot.hs).

## Documentation

The main challenge is finding a good location and zoom function to
reveals self-similar pattern.

Here is the less-minified version with type annotations:

```haskell
#!/usr/bin/env runhaskell
-- | tiny-brot expanded
-- Copyright 2023, Tristan de Cacqueray
-- SPDX-License-Identifier: CC-BY-4.0

-- | Render area.
w, h :: Float
w = 80
h = 24

-- | Screen ratio.
r :: Float
r = (-0.5)

-- | Compute z = z*z + c.
type Complex = (Float, Float)
z2 :: Complex -> Complex -> Complex
z2 (cx,cy) (x,y) = (cx + (x*x-y*y), cy + 2*x*y)

-- | The location of the tiny brot.
mb :: Complex
mb = (-1.4844, 0)

-- | Compute a smooth zoom.
type Time = Float
type Zoom = Float
zoom :: Time -> Zoom
zoom x = 4.18 - 4.179 * (1 - cos (x / 10) ** 8)

-- | Convert a screen coordinate to a plane coordinate.
type Coord = (Float, Float)
coord :: Zoom -> Complex -> Coord -> Complex
coord z (c,d) (x,y) = (c + z*(x-w/2)/w, (d + z*(y-h/2)/h)*r)

-- | Compute the length of a complex number.
dot :: Complex -> Float
dot (x,y) = let l = abs(x*y) in if isNaN l then 42 else l

-- | Iterate the mandelbrot function.
brot :: Int -> Complex -> Float
brot max_iter p = dot . last . take max_iter . iterate (z2 p) $ (0,0)

-- | Draw a single pixel.
d :: Zoom -> Coord -> String
d _ (81,_) = "\n"
d z c = if brot 150 (coord z mb c) > 20 then " " else "λ"

-- | The list of screen coordinate.
p :: [Coord]
p = [(x,y) | y <- [0..h], x <- [0..w+1]]

-- | Draw the fractal.
go :: (Time, Char) -> String
go (x, _) = "\ESCctiny-brot\n" <> concatMap (d (zoom x)) p

-- | Start the game, adding a time increment to each input.
main :: IO ()
main = interact (foldMap go . zip [00..] . mappend "g")
```
