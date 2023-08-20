# jsob-pils

(I haven't uploaded the source code for the language because it's too coupled into a game development project, at the moment.) 

Pils/JSOB: This is a combo project written in GDScript for the Godot game engine. Pils is a Lisp-like language that can be embedded into a Godot game. JSOB is a data declaration language, which is syntactically an S-expression. What is somewhat interesting is that JSOB can contain snippets of Pils, allowing for progammatic value assignment like Jsonnet; JSOB can also reference itself (via `@`), similar to YAML. I used JSOB as a DSL for an [incremental-style game](https://en.wikipedia.org/wiki/Incremental_game]), creating an engine that allows for an entire game to be defined in the DSL. For example, the DSL code (copied here from the file `witch.jsob` in this repo) below defines the game rendered in the following screenshot. Note the contrived example of `pils` on line 21, `(yield [@mLand (pils (if (> 1 0) 5 0))])`.


![jsob-demo](https://github.com/apsz3/jsob-pils/assets/62445385/f0ae96d3-6d98-44e6-8f9c-95f5f6132ff2)

The game logic is simple. Jobs are defined under the `jobs` heading, with a `j` prefix. Jobs take a certain time `t`, and have a `cost` in `materials` needed to start it. Jobs `yield` more `materials`, and are coupled with textual descriptions when `completed`. `actions` are like jobs, in that the player can activate them on the UI, but they don't cost or yield resources, but instead toggle game `state_id` changes, which can be used as predicates in situations such as `unlocks`.  

Notice how in the screenshot some jobs are greyed-out; this is because the `material` threshold isn't yet reached to activate them. Also notice that some actions and jobs defined in the DSL don't appear at all on the screen -- this is because the player's game state has not reached a point where the jobs/actions should be considered "unlocked", and available to the player to see.

```lisp
(module
    (materials
        (mLand (desc "The firmament"))
        (mMedicinalHerbs (desc "Medicinal herbs"))
				(mBerries (desc "Berries"))
				(mBranches (desc "Branches"))
				(mWater (desc "Water"))
				(mWood (desc "Wood")))
    (inventory
				(mLand 0) (mWood 0))
    (jobs
        (jClearTheLand
            (t 10)
						(yield [@mLand (pils (if (> 1 0) 5 0))])
            (desc (completed "Some land was cleared.") (display "Clear the Land")))
				(jForageTheWood
					(t 20) (cost [@mWater 1]) (requires [@mWater 1])
					(yield [[@mBerries {randi() % 15}] [@mMedicinalHerbs {randi() % 5}]])
					(desc
						(completed "A basketful of woodland bounty was collected")
						(display "Forage the Wood")))
				(jFetchWater
					(t 15)
					(yield [@mWater 1])
					(desc
						(completed "A bucket of water was fetched from the river")
						(display "Fetch Water")))
				(jSplitTheTrees
						(t 5)
						(yield [
							[@mBranches {2 + randi() % 5}]
							[@mWood {3 + randi() % 5}]])
						(desc (completed "Some trees were split")
									(display "Split the Trees"))))
		(actions
			(aPitchTents
				(state_id "pitch_tents")
				(t 10)
				(desc
					(completed "Tents were pitched in the clearing. More comfortable than sleeping in the Wagon.")
					(display "Pitch the tents")))
			(aBuildHome
				(state_id "build_home")
				(t 25) (cost [@mWood 50])
				(desc
					(completed "A home has been built.")
					(display "Build the Home"))))
(progression
    (unlocks [

        // init
        [@aPitchTents {Stockpile.mLand[0] >= 4}]
        [@jClearTheLand {State.ticks > 5}]

        // getting started
        [@jFetchWater {State.has("pitch_tents")}]
        [@jForageTheWood {State.has("pitch_tents")}]
        [@jSplitTheTrees {Stockpile.mLand[0] >= 9}]
        [@aBuildHome {Stockpile.mLand[0] >= 15}]

    ]) // --- /unlocks
    (message_events [
        // -------- Intro
        [|||Banished from their community for heresy, the Family has arrived after
            many days of travel at the edge of a great forest.||| {true}]
        [|||Gathered around the wagon, the man bows to the earth, consecrating the land; the woman, boy, and girl follow his example, touching soil to their lips.|||
            {State.ticks > 2}]
        ["The dog barks at the sound of the wind moving through the trees." {State.ticks > 4}]
        // -------- Initial
        ["Man and boy take axes and begin to cut at a leafy oak.\nThe women unload the wagon." {Stockpile.mLand[0] == 1}]
        ["The work is hard, but rewarding. A new life awaits." {Stockpile.mLand[0] == 2}]
        ["A small clearing is forming." {Stockpile.mLand[0] == 3}]

            ["Tents are pitched, temporary shelter while the home is built." {State.has("pitch_tents")}]
            ["There is still much ground to be broken: land for building and farming." {State.has("pitch_tents")}]
        // TODO -- we don't have a way of tying the state of an available action to the display for it being available...
        ["There is room enough for a cabin to be built now. The trees felled to make way shall provide the lumber, though they must be hewn and sawed." {Stockpile.mLand[0] >= 9}]
        ["A home has been built." {State.has("build_home")}]
])))
```
