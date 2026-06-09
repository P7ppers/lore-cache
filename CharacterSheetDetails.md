# Stride Lore-Cache: D&D Campaign Manager - Updated Character Sheet Schema
## Additions: Death Saves, CharacterSpellCasting, CharacterDescription, SpellLibrary/SpellBook

---

## Core Tables

### 1. Users (Supabase Auth)
```sql
auth.users (
  id UUID PRIMARY KEY,
  email TEXT UNIQUE,
  created_at TIMESTAMP,
  updated_at TIMESTAMP
)
```

---

### 2. Characters
```sql
CREATE TABLE characters (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
  
  name TEXT NOT NULL,
  class TEXT NOT NULL,
  race TEXT NOT NULL,
  subrace TEXT,
  background TEXT,
  alignment TEXT,
  
  level INT NOT NULL DEFAULT 1 CHECK (level >= 1 AND level <= 20),
  experience_points INT DEFAULT 0,
  
  created_at TIMESTAMP NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP NOT NULL DEFAULT NOW(),
  
  CONSTRAINT unique_character_per_user UNIQUE(user_id, name)
);

CREATE INDEX idx_characters_user_id ON characters(user_id);
```

---

### 3. AbilityScores
```sql
CREATE TABLE ability_scores (
  character_id UUID PRIMARY KEY REFERENCES characters(id) ON DELETE CASCADE,
  
  strength INT NOT NULL DEFAULT 10 CHECK (strength >= 1 AND strength <= 20),
  dexterity INT NOT NULL DEFAULT 10 CHECK (dexterity >= 1 AND dexterity <= 20),
  constitution INT NOT NULL DEFAULT 10 CHECK (constitution >= 1 AND constitution <= 20),
  intelligence INT NOT NULL DEFAULT 10 CHECK (intelligence >= 1 AND intelligence <= 20),
  wisdom INT NOT NULL DEFAULT 10 CHECK (wisdom >= 1 AND wisdom <= 20),
  charisma INT NOT NULL DEFAULT 10 CHECK (charisma >= 1 AND charisma <= 20),
  
  updated_at TIMESTAMP NOT NULL DEFAULT NOW()
);
```

---

### 4. CombatStats (UPDATED - Added Death Saves)
```sql
CREATE TABLE combat_stats (
  character_id UUID PRIMARY KEY REFERENCES characters(id) ON DELETE CASCADE,
  
  -- Core Combat
  ac INT NOT NULL DEFAULT 10,
  movement INT DEFAULT 30,
  hp_current INT NOT NULL DEFAULT 0,
  hp_max INT NOT NULL DEFAULT 1,
  temp_hp INT DEFAULT 0,
  
  -- Hit Dice
  hit_dice_current INT NOT NULL DEFAULT 0,
  hit_dice_total INT NOT NULL DEFAULT 0,
  hit_dice_type TEXT DEFAULT 'd8',
  
  -- Resources
  inspiration INT DEFAULT 0,
  attunement_slots INT DEFAULT 3,
  attunement_current INT DEFAULT 0,
  
  initiative_bonus INT DEFAULT 0,
  proficiency_bonus INT NOT NULL DEFAULT 2,
  
  -- Status conditions
  is_unconscious BOOLEAN DEFAULT FALSE,
  is_prone BOOLEAN DEFAULT FALSE,
  
  -- Death Saves (NEW)
  deathsave_successes INT DEFAULT 0 CHECK (deathsave_successes >= 0 AND deathsave_successes <= 3),
  deathsave_failures INT DEFAULT 0 CHECK (deathsave_failures >= 0 AND deathsave_failures <= 3),
  
  -- Class-specific resources (JSON)
  spec_features JSONB DEFAULT '{}',
  
  updated_at TIMESTAMP NOT NULL DEFAULT NOW(),
  
  CONSTRAINT valid_hp CHECK (hp_current >= 0 AND hp_current <= hp_max),
  CONSTRAINT valid_temp_hp CHECK (temp_hp >= 0),
  CONSTRAINT valid_inspiration CHECK (inspiration >= 0)
);
```

---

### 5. CharacterDescription (NEW)
```sql
CREATE TABLE character_description (
  character_id UUID PRIMARY KEY REFERENCES characters(id) ON DELETE CASCADE,
  
  -- Character Bio Data (JSON)
  -- Structure: { age: number, eye_color: string, height: string, weight: string, 
  --              skin_color: string, hair_color: string, misc_features: string }
  character_bio_data JSONB DEFAULT '{}',
  
  -- Character Appearance (JSON)
  -- Structure: { image_url: string, description: string }
  character_appearance JSONB DEFAULT '{}',
  
  -- Backstory and personality
  character_backstory TEXT,
  allies TEXT,                         -- Names/descriptions of allied NPCs
  
  -- D&D 5e Personality/Background traits (JSONB arrays for flexibility)
  personality_traits JSONB DEFAULT '[]',  -- Array of strings (e.g., ["I'm always eating", "I talk with my hands"])
  ideals JSONB DEFAULT '[]',               -- Array of strings (e.g., ["Freedom. People should be free to pursue their own goals"])
  bonds JSONB DEFAULT '[]',                -- Array of strings (e.g., ["I owe a life debt to my companion"])
  flaws JSONB DEFAULT '[]',                -- Array of strings (e.g., ["I'm reckless to the point of self-harm"])
  
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);
```

**character_bio_data example:**
```json
{
  "age": 28,
  "eye_color": "blue",
  "height": "6'0\"",
  "weight": "180 lbs",
  "skin_color": "fair",
  "hair_color": "brown",
  "misc_features": "Scar across left cheek, tattoo of a dragon on right arm"
}
```

**character_appearance example:**
```json
{
  "image_url": "https://s3.example.com/characters/my-char-portrait.jpg",
  "description": "A rugged human with weathered features and keen eyes. Wears well-maintained leather armor and carries a well-worn longsword."
}
```

**personality_traits example:**
```json
[
  "I'm always eating and talking with my mouth full",
  "I drum my fingers constantly on any surface"
]
```

**ideals example:**
```json
[
  "Freedom. People should be free to pursue their own goals",
  "Honor. I must live up to my word and actions"
]
```

**bonds example:**
```json
[
  "I owe a life debt to the ranger who saved my life",
  "My lost sister is somewhere out there, and I will find her"
]
```

**flaws example:**
```json
[
  "I'm reckless to the point of self-harm",
  "I have a terrible temper when insulted"
]
```

---

### 6. CharacterSpellCasting (NEW)
```sql
CREATE TABLE character_spellcasting (
  character_id UUID PRIMARY KEY REFERENCES characters(id) ON DELETE CASCADE,
  
  spellcasting_ability TEXT NOT NULL,  -- "strength", "dexterity", "constitution",
                                       -- "intelligence", "wisdom", "charisma"
  spell_save_dc INT NOT NULL,
  spell_attack_bonus INT NOT NULL,
  
  updated_at TIMESTAMP NOT NULL DEFAULT NOW(),
  
  CONSTRAINT valid_ability CHECK (
    spellcasting_ability IN ('strength', 'dexterity', 'constitution',
                             'intelligence', 'wisdom', 'charisma')
  )
);
```

---

### 7. CharacterFeatures (Denormalized)
```sql
CREATE TABLE character_features (
  character_id UUID NOT NULL REFERENCES characters(id) ON DELETE CASCADE,
  feature_name TEXT NOT NULL,
  
  feature_type TEXT NOT NULL,
  source_class TEXT,
  source_subclass TEXT,
  source_race TEXT,
  source_background TEXT,
  
  description TEXT NOT NULL,
  short_description TEXT,
  prerequisite TEXT,
  level_gained INT,
  
  is_proficiency BOOLEAN DEFAULT FALSE,
  proficiency_type TEXT,
  
  notes TEXT,
  is_active BOOLEAN DEFAULT TRUE,
  proficiency_level INT DEFAULT 1,
  
  created_at TIMESTAMP DEFAULT NOW(),
  
  PRIMARY KEY (character_id, feature_name),
  
  CONSTRAINT valid_feature_type CHECK (
    feature_type IN ('Class Feature', 'Feat', 'Racial Trait', 'Background Feature',
                     'Language', 'Proficiency', 'Subclass Feature')
  ),
  CONSTRAINT valid_proficiency_level CHECK (proficiency_level IN (0, 1, 2))
);

CREATE INDEX idx_character_features_type ON character_features(feature_type);
```

---

### 8. SpellLibrary (RENAMED from Spells - UPDATED with description)
```sql
CREATE TABLE spell_library (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  
  name TEXT NOT NULL UNIQUE,
  spell_level INT NOT NULL CHECK (spell_level >= 0 AND spell_level <= 9),
  school TEXT NOT NULL,
  
  casting_time TEXT,
  range TEXT,
  duration TEXT,
  components TEXT,
  description TEXT,  -- NEW: Full spell mechanics and effects
  
  ritual BOOLEAN DEFAULT FALSE,
  concentration BOOLEAN DEFAULT FALSE,
  
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_spell_library_level ON spell_library(spell_level);
CREATE INDEX idx_spell_library_school ON spell_library(school);
```

---

### 9. SpellBook (RENAMED from CharacterSpells)
```sql
CREATE TABLE spell_book (
  character_id UUID NOT NULL REFERENCES characters(id) ON DELETE CASCADE,
  spell_id UUID NOT NULL REFERENCES spell_library(id) ON DELETE CASCADE,
  
  is_prepared BOOLEAN DEFAULT FALSE,
  is_known BOOLEAN DEFAULT TRUE,
  spell_slot_level INT DEFAULT 1,
  
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  
  PRIMARY KEY (character_id, spell_id)
);

CREATE INDEX idx_spell_book_spell_id ON spell_book(spell_id);
```

---

### 10. CharacterItems (Denormalized)
```sql
CREATE TABLE character_items (
  character_id UUID NOT NULL REFERENCES characters(id) ON DELETE CASCADE,
  item_name TEXT NOT NULL,
  
  rarity TEXT,
  type TEXT,
  
  weight DECIMAL(10, 2),
  cost TEXT,
  description TEXT,
  
  damage_dice TEXT,
  damage_type TEXT,
  ac_bonus INT DEFAULT 0,
  
  is_magical BOOLEAN DEFAULT FALSE,
  
  quantity INT NOT NULL DEFAULT 1 CHECK (quantity > 0),
  is_equipped BOOLEAN DEFAULT FALSE,
  is_attuned BOOLEAN DEFAULT FALSE,
  notes TEXT,
  
  PRIMARY KEY (character_id, item_name)
);

CREATE INDEX idx_character_items_type ON character_items(type);
```

---

### 11. Notes (Unchanged)
```sql
CREATE TABLE notes (
  character_id UUID PRIMARY KEY REFERENCES characters(id) ON DELETE CASCADE,
  
  content TEXT,
  created_at TIMESTAMP NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP NOT NULL DEFAULT NOW()
);
```

---

## Implementation Order

1. Create **Characters, AbilityScores**
2. Create **CombatStats** (with death saves)
3. Create **CharacterDescription** (new)
4. Create **CharacterSpellCasting** (new)
5. Create **CharacterFeatures**
6. Create **SpellLibrary** (renamed from Spells)
7. Create **SpellBook** (renamed from CharacterSpells)
8. Create **CharacterItems**
9. Create **Notes**

---

## Summary of Changes

| Change | What | Why |
|--------|------|-----|
| **CombatStats** | Added deathsave_successes, deathsave_failures | Track death saving throws |
| **New Table** | CharacterSpellCasting | Centralize spellcasting mechanics (ability, DC, attack bonus) |
| **New Table** | CharacterDescription | Store character appearance, bio, backstory, relationships, personality |
| **CharacterDescription** | Added personality_traits, ideals, bonds, flaws (JSONB arrays) | Store D&D 5e personality traits in structured format |
| **Renamed** | Spells → SpellLibrary | Clarity: library of all available spells |
| **Renamed** | CharacterSpells → SpellBook | Clarity: character's personal spell selection |
| **SpellLibrary** | Added description field | Store full spell mechanics and effects |

---

## Future Enhancements

### CSV Import for Spell Lists
Once core features are live, add the ability to:
1. Upload CSV files with class spell lists (Druid, Artificer, Cleric, etc.)
2. Bulk-insert spells into SpellLibrary
3. Let users select which class spells to add to their character's SpellBook
4. Option to add custom spells manually

**CSV format example:**
```csv
name,spell_level,school,casting_time,range,duration,components,description
Magic Missile,1,Evocation,1 action,120 feet,Instantaneous,"V, S","You hurl magical darts..."
Mage Armor,1,Abjuration,1 action,Touch,"8 hours","V, S, M (piece of leather)","You touch a willing creature..."
```

---

## Database Diagram Summary

```
Users (Supabase)
  ↓
Characters ←─→ AbilityScores
  ├─→ CombatStats (+ death saves + spec_features)
  ├─→ CharacterDescription (+ bio + appearance + backstory)
  ├─→ CharacterSpellCasting (+ spell mechanics)
  ├─→ CharacterFeatures (all character features/skills/proficiencies)
  ├─→ SpellBook ←─→ SpellLibrary (character's spells)
  ├─→ CharacterItems (inventory)
  └─→ Notes (session notes)
```

All tables are properly indexed and constrained for data integrity.
