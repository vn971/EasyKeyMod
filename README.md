Hey folks!  
I'm thinking to start a new project for keyboard configuration and I'd love your feedback.

Please join the [Matrix channel](https://matrix.to/#/#EasyKeyMod:matrix.org) (or [Discord](https://discord.gg/EzZc8rFn) for those who cannot join Matrix),
and read on to provide any feedback you might have.

# Why would you be interested?
* I think I have an idea for a "[home row mod](#Home-row-mod)" that may reduce your typos happening from key rollover.
* I think I have an idea how to make the project [last really long for you](#This-project-can-last-you-for-long). Without getting obsolete and forcing you to re-learn all the time.
* I think I have an idea how to allow *you* decide how your keyboard works, [without waiting on feature requests](#You-decide-everything) from me.

# Home row mod
I could be wrong on this section, so if you have any concerns, please share.

The idea is that many of the current home-row mods are prone to roll-over, OR you have to wait to convert your key-tap to a key-hold. And if you want to press multiple key modifiers, then you may have to press the first one, wait, press the second one, wait, and only then finally press the actual key.

The alternative idea is to allow definitinos like "that (modifier) key and the target key were pressed for more than 50ms at the same time". Do you see the subtle but important difference to "the first key was already held for some time and we've just pressed down Y"?
```txt
[---------- home row key ----------]
                                   [----- normal key -----]
                                   ^--- Home row in Vial will activate

[------- home row key -------]
                [----- normal key -----]
                <--50ms-->
                       ^--- Proposed activation rule


[----- home row key -----]
  [----- home row key -----]
                     [----- normal key -----]
  ^--- Vial breaks here, outputting 2 letters instead
                     ^--- and a third one here

[------- home row key -------]
  [------- home row key -------]
                  [----- normal key -----]
                  <--50ms-->
                           ^--- Proposed activation rule
```

And it'd be possible to further customize it if desired, e.g. that one button was pressed earlier than the other. Or with certain exclusion/inclusion rules as e.g. can be done in `kanata`. Fully customizable if/when you need.

# You decide everything
This idea isn't new.  
For example, QML lets you program your keyboard however you want. There are some problems with that though. The project is ridden with forks, e.g. a fork for Miryoku, a fork for my new keyboard X, etc. It also requires compilation (on ArchLinux the packages for QML are over 300Mb - ouch, really?). And you have a smart system running on your goddamn keyboard; that may not exactly relax those security-conscious people among us.

Another example  
where the user is let to decide a lot, if not everything, is `kanata`. Indeed, they have `if` conditions and `switch` cases [baked in their custom configuration language](https://github.com/jtroo/kanata/blob/main/docs/config.adoc#switch). That's nice, but if you want something different than their out-of-the-box features, you have to potentially fork and deal with changes accross Rust + their configuration language + your own config + documentation. And that's if your change is not major enough to call for a re-factor.

### So what is the proposal?

I propose to use a simple, embeddable and secure language. In which it would be easy to express
all the rules, yet where you cannot execute/read random stuff on the computer. Well, we already know that language, it's called Lua. Example configuration:
```lua
{ "layer_while_held", "numbers" }

layer_while_held("numbers")

layer_while_held { numbers, timeout = 12 }
```

Which of these variatinos is actually the correct one? Well it's not my decision, it's yours. The engine will only expose the required information to your Lua code:

* what is the trigger event
* any previously stored variables

With this information, you can do virtually anything.
Of course, the first version of EasyKeyMod will ship with some Lua code that has a bunch of those functions defined, so that you can get started with just a bit of configuration. But with the choice of Lua, we won't force it on you -- we'll only allow you importing some community examples code.

# This project can last you for long
We believe that this is the implication of the above. The actual keyboard engine will only:

* read keyboard events
* let user code run, to produce actual output
* (allow the user code to place wake-up calls for timeouts)

That is not that hard to do. Any code created this way would also be trivial to import to a competitor engine.

I think it's a little bit like the Unix philosophy, where you do one thing but do it well.
If you know that the basis for your keyboard configuration is simple and only responsible for one part, you're less afraid about it's failure or complexity overload. And if a community config got too complicated for whatever reason, you can choose to not import it and use your own code.

# Example
Here is an example configuration for maybe some out-of-the-box thinking(?):
```lua
-- Rules are evaluated from top to bottom. First matching rule wins.

-- The default arguments are skipped as it's a function I've defined above.  
combo("a", "b", "z")  -- if "a" and "b" are pressed together, output "c" instead.

-- A rule applied after the more complex ones above but before the more smiple general rules
sticky_modifier("shift", 150) -- the good old sticky Shift key

function pr(letter)
  return pressed_for_milliseconds(letter, 50)
end

rule { layer = "numbers", pr("h"), "6")

rule { layer = basic_layers_only, pr("a"), pr("b"), "z")

local a = "KC_A" -- ... other keys too

define_layer {
  name = "numbers",
  Fn1, Fn2, Fn3
  1,   2,   3
}
```

# Thoughts?
Thoughts? Ideas? Just want to discuss something on this? Please reach out in Matrix / Discord!  
If this project is to be active or even exist, it's important to have people with different perspectives. Including different from any early authors such as myself. The project above is already described with the mindset of having many approaches and points of view, so it's important to have this community mindset right from the beginning.

# FAQ
Q: What if I want multi-tap? Will this be possible?  
A: Yes. You'll need to store information on the timings of previous key presses/releases and use it later. Obviously, there will be some functions provided that you can import out-of-the-box. Or you could not import them and write your own logic.

Q: Will there be Macros?  
A: Sure, in any way you like it.

Q: The project has little mentions of layers. Will they be natively supported by the engine?  
A: There's no need for native support, as you can just keep a variable around and have your event processing use that variable.

Q: Will it be difficult to write rules in this strange Lua of yours?  
A: We believe not. Most of the functionalities by `kanata` / `Kmonad` is really straight-forward and easy, once you have the raw events coming. The bigger questions is usually on documentation, choosing what to implement and what _not_ to implement, making defaults to target a large set of desires but niche enough to include what you want. By comparison, a layout author to EasyKeyMod would have to learn a single language only, and it will work now and on any future versions of the engine.

Q: Will you improve this FAQ, more than iterating three times over a single feature?  
A: Yes... But do ask other questions first!
