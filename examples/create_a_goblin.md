Creating goblins could be as simple as adding some statistics inside a JSON file and calling it a day, but mechanic allows you to create goblins with randomization elements and customize every type of goblin there is.

For example, imagine we want to generate a troup of 10 goblins, with a non-equal distribution of sword weilding goblins, bow weilding goblins and goblin shamans. Since we don't want our level 1 adventurers to be destroyed, we set our weight to a d10 roll. On a 1 to 6, it's a Sword goblin, on a 7 to 9 it's a bow goblin and on a 10 it's a shaman.

```
import MonsterFactory from '...'
import SpellFactory from '...'

const int SWORD_GOBLIN = 1
const int BOW_GOBLIN = 2
const int GOBLIN_SHAMAN = 3

begin factory GoblinFactory from MonsterFactory
    begin produce
        int health
        float xpGain
        int type
    end produce
    
    @main
    begin step main(GoblinFactory.produce initial)
        initial.type = list([
            0..5: SWORD_GOBLIN,
            6..8: BOW_GOBLIN,
            GOBLIN_SHAMAN,
        ])[randomize(0, 9)]
        return initial to GoblinFactory.swordGoblin, GoblinFactory.bowGoblin, GoblinFactory.goblinShaman
    end step
    
    @condition(type == SWORD_GOBLIN)
    begin step swordGoblin(GoblinFactory.produce entity)
        entity.health = randomize(5, 8)
        entity.xpGain = entity.health * 10
        return entity
    end step
    
    @condition(type == BOW_GOBLIN)
    begin step bowGoblin(GoblinFactory.produce entity)
        entity.health = randomize(6, 9)
        entity.xpGain = entity.health * 10
        return entity
    end step
    
    @condition(type == GOBLIN_SHAMAN)
    begin step goblinShaman(GoblinFactory.produce entity)
        entity.health = randomize(6, 9)
        entity.xpGain = entity.health * 10
        return GoblinShamanFactory.newFrom(entity) // Still unsure about this, maybe using a classic 'super' would be better?
    end step
end factory

begin factory GoblinShamanFactory from GoblinFactory
    begin produce
        string name
        int health
        float xpGain
        int type
        SpellFactory.produce[] spells
    end produce
    
    @main
    begin step main(GoblinFactory.produce initial)
        // Initial should already be set
        initial.spells = list()
        int numberOfSpells = randomize(1, 3)
        for (int i = numberOfSpells; i > 0; i -= 1) loop
            initial.spells = list([..initial.spells, SpellFactory.new("goblin")] //Let's just imagine spell factory can make specific spells
        end for
        return initial
    end step
end factory

begin function createGoblin() returns (GoblinFactory.produce, error)
    return GoblinFactory.new(), empty
end function
```
