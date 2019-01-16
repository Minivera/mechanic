# How to create a quest using mechanic
Creating a random quest may require some external data for language and text, but it can also be simplified to a simple step process. These quests will expect the player to move somewhere in the world, eliminate monsters, collect an item or talk to a NPC then return.

1. Find the quest giver
2. Find the location of the objective
3. Find the objective
4. Generate the objective

```
import NPCFactory from '...';
import LocationFactory from '...';
import EntityFactory from '...';
import MonsterFactory from '...';
import ItemFactory from '...';

const int KILL_QUEST = 1
const int FETCH_QUEST = 2
const int TALK_QUEST = 3

begin factory QuestFactory uses (NPCFactory.produce[] npcs, LocationFactory.produce[] locations)
    begin produce
        int difficulty
        int reward
        int questType
        string questLine
        LocationFactory.produce location
        NPCFactory.produce questGiver
        EntityFactory.produce[] objective
    end produce
    
    @main
    begin step main(QuestFactory.produce initial)
        initial.difficulty = randomize(1, 5)
        initial.reward = 500 * initial.difficulty
        initial.questType = randomize(1, 3)
        initial.location = self.locations[randomize(0, len(self.locations) - 1)]
        initial.questGiver = self.npcs[randomize(0, len(self.npcs) - 1)]
        return initial to QuestFactory.killQuest, QuestFactory.fetchQuest, QuestFactory.talkQuest
    end step
    
    @condition(questType is KILL_QUEST)
    begin step killQuest(QuestFactory.produce quest)
        quest.objective = list()
        string names = "";
        for (int i < quest.difficulty) loop
            quest.objective[i] = MonsterFactory.new()
            names += "quest.objective[i].name, "
        end for
        
        names -= ", ";
        quest.questLine = "Defeat " + names + " at " + initial.location.name
        return quest
    end step
    
    @condition(questType is FETCH_QUEST)
    begin step fetchQuest(QuestFactory.produce quest)
        ItemFactory.produce item = ItemFactory.new()
        quest.objective = list()
        for (int i < quest.difficulty) loop
            quest.objective[i] = item
        end for
        quest.questLine = "Find " + difficulty + " " + item.name +  " at " + initial.location.name
        return quest
    end step
    
    @condition(questType is TALK_QUEST)
    begin step talkQuest(QuestFactory.produce quest)
        NPCFactory.produce npc = self.npcs[randomize(0, len(self.npcs) - 1)]
        quest.objective = list([npc])
        quest.questLine = "Talk to " + npc.name +  " at " + initial.location.name
        return quest
    end step
end factory

begin function createQuest(NPCFactory.produce[] npcs, LocationFactory.produce[] locations) 
    returns (QuestFactory.produce, error)
    return QuestFactory.new(npcs, locations), empty
end function
```
