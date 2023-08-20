# jsob-pils

The DSL code below defines all the behavior and game progression logic seen in this screenshot. The file is `witch.jsob`.

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
