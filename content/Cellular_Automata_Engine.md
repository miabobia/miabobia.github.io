---
title: Cellular Automata Engine
draft: false
tags:
    - simulations
    - pygame
    - system design
    - model view controller
    - art
    - image-processing
    - open-cv
    - python
    - PIL
---

# [Repository](https://github.com/miabobia/cellular_automata_engine/tree/mvc_branch)

# What is a Cellular Automata?
You've probably heard of [Conway's Game of Life](https://en.wikipedia.org/wiki/Conway%27s_Game_of_Life)

![[https://miabobia.github.io/conways_example.gif]]

Conway's Game of Life is actually just one of Cellular Automata that exist! Cellular Automata can have many different rulesets (rules on conditions for cell's to come alive, survive or die). What I made is an engine that can support **any** ruleset for Cellular Automata. It also is setup modularly so the Automata can be rendered in many different ways

# Why?
This idea has been fully explored by others at this point, but Cellular Automata are something I find very exciting. So, I took this *solved* problem as an opportunity to learn about software design patterns. I ended up writing my engine using three different design patterns:
- Observer Viewer Model
- Event Emitters/Subscribers
- Model View Controller

# Technologies I used
- python
- pygame
- PIL
- opencv

# [Observer Viewer Model](https://github.com/miabobia/cellular_automata_engine/commit/4fc94b47cbb29aa58239e3872888ba544342874e#diff-b10564ab7d2c520cdd0243874879fb0a782862c3c902ab535faabe57d5a505e1R100-R101)
The implementation had a Grid that handled all the logic of the automata and the Grid had a viewer attached to it. The idea was that whenever the grid had a change in state that it would notify its viewer to render the grid.

**grid**
```python
@dataclass
class Grid:
    width: int
    height: int
    cells: List[Cell]
    ...
    def add_observer(self, _observer) -> None:
        self.observer = _observer
    def notify_observer(self) -> None:
        self.observer.update()
```

**main game loop**
```python
while True:
    ...
    grid.calculate_next_generation()
    grid.notify_observer()
```

while this approach *worked* it didn't feel quite right. It had issues with the chain of command. Say I wanted to update the pallete for the viewer or make any change that would only effect the viewer and not the state of the game. How would I do this? I'd have to give instructions to the Grid that would then be passed down to the viewer. This felt clunky and I wasn't satisfied with this approach. 

# [Event Emitters/Subscribers](https://github.com/miabobia/cellular_automata_engine/commit/3b0a2837f455038756784d7ec72db25963ee56e4)
After a suggestion from a friend I tried to change up my architecture to have all my components communicate through event listeners and emmitters

**event classes**
```python
from typing import Any, Callable
from dataclasses import dataclass
@dataclass
class Event:
    name: str
    data: Any
class EventDispatch:
    listeners = {}
    def add_listener(self, name: str, listener: Callable) -> None:
        if name not in self.listeners:
            self.listeners[name] = []
        self.listeners[name].append(listener)
    
    def dispatch(self, event: str) -> None:
        if event.name not in self.listeners: return
        for listener in self.listeners[event.name]:
            listener(event)
```

**how Viewer used the events system**
```python
class Viewer:
    def __init__(self, _screen: pg.surface, _display_config: DisplayConfig, _event_dispatch: EventDispatch):
        self.event_dispatch = _event_dispatch
        self.screen = _screen
        self.screen_size = self.screen.get_size()
        self.display_config = _display_config
        self.pallete = self.display_config.data["pallete"]

    def set_model(self, _model: model.Model) -> None:
        self.event_dispatch.add_listener("update_pallete", self.update_pallete)
```

**how other classes would dispatch to Viewer and update its pallete**
```python
self.event_dispatch.dispatch(Event("update_pallete", None))
```

This approach was fun to implement, but after rubber ducking with a friend I realized it didn't suit my problem. There were only singletons of my components and having them listen and emit to eachother was effectively the same as them all having references to eachother and calling functions on one another. Very fun design pattern to learn about and i'm excited to use it when I have a problem that better suits it :)

# Model View Controller
Ultimately this approach felt the most sensible to me. My friend Heather and I whiteboarded out a diagram for how I would design this sytem

![[system_diagram.png|250]]

In this setup I had these components:
## Model
- contains the Grid class which handles the logic of the automata
- has a function called `step()` which is the main *loop* of the application
```python
def step(self) -> bool:
    """
    takes a step in the game loop
    grid -> calculates next generation
    viewer -> tells viewer to render
    """
    # model tells grid and viewer to update
    # returns true if there are more simulations to run
    if self.generation == self.total_generations:
        self.viewer.cleanup()
        return False
    if self.running:
        self.generation += 1
        self.grid_model.calculate_next_generation()
    self.viewer.update(self.grid_model)
    return True
```
step tells the **Viewer** when to render and the **Grid** when to calculate its next generation

## Display Config
- contains config information/functions to handle config attributes purely pertaining to the **Viewer** class
```python
class DisplayConfig:
    pallete_set = [
        palletes.ClassicPallete, palletes.TransPallete,
        palletes.MatrixPallete, palletes.RetroPallete,
        palletes.GameBoyPallete, palletes.PastelPinkYellowPallete,
        palletes.PastelBlueYellowPallete, palletes.BlackRedPallete
    ]

    data = {
        "pallete_index": 0,
        "pallete": palletes.ClassicPallete
    }
...
    def update_pallete_index(self, n: int) -> None:
        # update pallete index and current pallete 
        self.data["pallete_index"] += n
        if self.data["pallete_index"] >= len(self.pallete_set):
            self.data["pallete_index"] = 0
        elif self.data["pallete_index"] < 0:
            self.data["pallete_index"] = len(self.pallete_set) - 1
        
        self.data["pallete"] = self.pallete_set[self.data["pallete_index"]]
```

## Viewer
- a viewer can be anything. Currently there are two types of **Viewers**
    - ExportView -> exports the automata simulation to an mp4 file
    - GridView -> uses pygame to render the automata in real time and supports interactability with **Controller** component
- its main source of truth is the **Display Config** component
- for example this is how it updates its pallete
```python
def update_pallete(self):
    """
    controller will send signal to display config to
    update the pallete. display config will send signal to
    viewer to read new pallete from display config 
    """
    self.pallete = self.display_config.data["pallete"]
```
- **Model** component informs it when to update
- **Controller** instructs it when to update config settings

## Controller
- has the most *power* of all the components. It can talk to the model, viewer and display config when needed
- informs the **Viewer** and **Model** when to make changes based on user input
- most of its functionality is contained in the `read_input()` function. This snippet of `read_input()` is how the pallete is updated for the **Viewer** 
```python
def read_input(self):
    for event in pygame.event.get():

        # key press event handling
        if event.type == pygame.KEYDOWN:
            # `d` increments the pallete index
            if event.key == pygame.K_d and not self.key_pressed["d"]:
                self.key_pressed["d"] = True
                self.config.update_pallete_index(1)
                self.viewer.update_pallete()

            # `s` decrements the pallete index
            elif event.key == pygame.K_s and not self.key_pressed["s"]:
                self.key_pressed["s"] = True
                self.config.update_pallete_index(-1)
                self.viewer.update_pallete()
    ...
```

# Future Plans

# Example Videos
Here are just a few examples of the automata you can generate with an ExportViewer!

Ruleset -> [Conway's Game of Life](https://conwaylife.com/wiki/Conway%27s_Game_of_Life)<br>
Grid Size -> 500 x 500<br>
Pallete -> Matrix<br>
Lifetimes -> False<br>
Generations -> 150<br>
![[ConwayBigMatrixGif.gif|750]]

Ruleset -> [High Life Ruleset](https://conwaylife.com/wiki/OCA:HighLife)<br>
Grid Size -> 500 x 500<br>
Pallete -> Black Red<br>
Lifetimes -> True<br>
Generations -> 300<br>
![[HighLifeWithLifetimes.gif|750]]

Ruleset -> [Day and Night Ruleset](https://conwaylife.com/wiki/OCA:Day_%26_Night)<br>
Grid Size -> 250 x 250<br>
Pallete -> Cycles<br>
Lifetimes -> True<br>
Generations -> 300<br>
![[DayAndNight.gif|750]]
