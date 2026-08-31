# Arsenal & Entity Catalogs

Making your items obtainable in game: faction arsenals, supply costs, rank gates.

## The mechanism

Faction inventories are **entity catalog configs**
(`Configs/EntityCatalog/<Faction>/InventoryItems_EntityCatalog_<Faction>.conf` for US, USSR, FIA,
CIV, FactionLess). You add items by **overriding the vanilla catalog by GUID**
([Config overrides](../50-scripting/config-overrides.md)) and appending your own list entry:

```
SCR_EntityCatalogMultiList {
 m_aMultiLists + {                              // "+" is MANDATORY — see below
  SCR_EntityCatalogMultiListEntry "{NEW-GUID}" {
   m_sIdentifier "MyModItems"                   // free dev label; all lists get merged
   m_aEntities {
    SCR_EntityCatalogInventoryItem "{NEW-GUID}" {
     m_sEntityPrefab "{YOUR-GUID}Prefabs/.../MyItem.et"
     m_aEntityDataList {
      SCR_ArsenalItem "{NEW-GUID}" {
       m_eItemType RIFLE                        // SCR_EArsenalItemType: RIFLE, HEADWEAR,
                                                // WEAPON_ATTACHMENT, AMMUNITION, …
       m_eItemMode DEFAULT                      // DEFAULT / WEAPON / ATTACHMENT / …
       m_iSupplyCost 20
       m_eRequiredRank SERGEANT                 // optional; default PRIVATE
      }
     }
    }
   }
  }
 }
}
```

- ⚠️ **`m_aMultiLists +`** — without the `+` your block *replaces* the vanilla array and the
  faction's arsenal contains only your item.
- `m_sIdentifier` is a free label — the catalog init merges **all** multi-lists regardless of
  name; you don't need to match vanilla list names.
- Item type + mode must fit the item (weapon vs attachment vs headwear…): a wrong pair hides the
  entry.
- Catalogs load at game start → test in a fresh session.

## Rank gates

`m_eRequiredRank` (PRIVATE < CORPORAL < SERGEANT < LIEUTENANT < CAPTAIN < MAJOR) applies **only
in game modes with a rank system** (Conflict). In a rankless test scenario everyone counts as
Private → your MAJOR-gated item looks missing/locked.
**When "item missing from arsenal", check the game mode before debugging the catalog.**

## Why your item still doesn't show up

Checklist from real debugging:

1. Catalog meta carries the **vanilla GUID**? (fresh GUID = dead override — the #1 cause)
2. `m_aMultiLists +` present?
3. Item prefab derived from a proper vanilla equipment base? Standalone items lack the correct
   inventory-item component subclass, the loadout area, and the **faction affiliation
   component** — any of which keeps them out of faction arsenals. Duplicate the closest vanilla
   item and attach your components instead.
4. Rank gate + game mode (above)?
5. Multiple loaded mods overriding the same catalog GUID merge by their entry instance GUIDs —
   normally they stack; when testing conflicts, load your mod alone first.

## Custom arsenal crates

Instead of touching faction catalogs: place a crate with `SCR_ArsenalComponent` and set its
*Overwrite Arsenal Config* to your own `SCR_ArsenalItemListConfig` listing your items per mode —
faction-independent, scenario-local. Fixed stock counts: a universal inventory storage with
multi-slots.

## Supply economy notes

- `m_iSupplyCost` matters only where supplies are enabled (Conflict). Game Master has the global
  supply economy off — items are free there.
- Script-side resource consumption (for custom build/purchase systems) goes through the resource
  component's consumer interface; read availability, request consumption, check the response —
  and expect it to be a no-op in economy-less modes.
