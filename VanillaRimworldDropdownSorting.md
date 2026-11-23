# Vanilla Rimworld Dropdown Sorting

`DesignationCategoryDef.ResolveDesignators()` (Comments added)

```C#
private void ResolveDesignators()
{
    resolvedDesignators.Clear();
    foreach (Type specialDesignatorClass in specialDesignatorClasses)
    {
        Designator designator = null;
        try
        {
            designator = (Designator)Activator.CreateInstance(specialDesignatorClass);
            designator.isOrder = true;
        }
        catch (Exception ex)
        {
            Log.Error("DesignationCategoryDef" + defName + " could not instantiate special designator from class " + specialDesignatorClass?.ToString() + ".\n Exception: \n" + ex.ToString());
        }
        if (designator != null)
        {
            resolvedDesignators.Add(designator);
        }
    }

    // TerrainDefs and thingDefs concatenated and filtered by designationCategory and canGenerateDefaultDesignator
    IEnumerable<BuildableDef> enumerable = from tDef in DefDatabase<TerrainDef>.AllDefs.Cast<BuildableDef>().Concat(DefDatabase<ThingDef>.AllDefs.Cast<BuildableDef>())
        where tDef.designationCategory == this && tDef.canGenerateDefaultDesignator
        select tDef;

    // Dropdown compiling
    Dictionary<DesignatorDropdownGroupDef, Designator_Dropdown> dictionary = new Dictionary<DesignatorDropdownGroupDef, Designator_Dropdown>();
    foreach (BuildableDef item in enumerable)
    {
        if (item.designatorDropdown != null)
        {
            // Create the dropdown if it doesn't exist yet
            if (!dictionary.ContainsKey(item.designatorDropdown))
            {
                dictionary[item.designatorDropdown] = new Designator_Dropdown();
                dictionary[item.designatorDropdown].Order = item.uiOrder;
                resolvedDesignators.Add(dictionary[item.designatorDropdown]);
            }

            // Add the designator to the dropdown
            dictionary[item.designatorDropdown].Add(new Designator_Build(item));
        }
        else
        {
            resolvedDesignators.Add(new Designator_Build(item));
        }
    }
}
```

`Designator_Dropdown.Add()`

```C#
public void Add(Designator des)
{
    elements.Add(des);
    if (activeDesignator == null)
    {
        SetActiveDesignator(des, explicitySet: false);
    }
}
```

SetupFloatMenu() then iterates `elements` by index. No sorting is done.