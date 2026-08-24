# Icons

## Built-in icons

StellarAdmin ships the [Lucide](https://lucide.dev/) icon set. It is registered by `AddTagHelpers()`, so every Lucide icon is available from `<sa-icon>` by name with no further setup:

```razor
<sa-icon name="plane"/>
```

- Names are the kebab-case Lucide names (`circle-arrow-left`, `layout-dashboard`, `map-pinned`), matched **case-insensitively**.
- When no icon matches, `<sa-icon>` renders a placeholder "not found" icon rather than nothing, so a typo is visible on the page rather than silent.
- Icons render as inline `<svg>` using `currentColor`, so they take the color and size of their surroundings. Size and color them with `class` (`size-5`, `text-muted-foreground`), not with attributes. `stroke-width` passes through to the `<svg>`.

Icons compose naturally inside other components:

```razor
<sa-button><sa-icon name="plus"/>New booking</sa-button>
```

## Adding your own icons

Custom icons are registered on the builder returned by `AddTagHelpers()`, then used exactly like the built-in ones. An icon is an `IconDefinition`: the attributes for the `<svg>` element plus the ordered list of shapes (`path`, `circle`, `rect`, …) drawn inside it.

Draw on the same 24 x 24 grid with a 2 px `currentColor` stroke to stay visually consistent with Lucide. Any attribute you put on `<sa-icon>` (such as `class` or `stroke-width`) takes precedence over the definition's attributes.

### A single icon — `AddIcon`

```csharp
using System.Collections.Immutable;
using StellarAdmin.TagHelpers.Icons;

builder.Services.AddStellarAdmin()
    .AddTagHelpers()
    .AddIcon("voyager-suitcase", new IconDefinition(
        new Dictionary<string, string>
        {
            ["xmlns"] = "http://www.w3.org/2000/svg",
            ["width"] = "24",
            ["height"] = "24",
            ["viewBox"] = "0 0 24 24",
            ["fill"] = "none",
            ["stroke"] = "currentColor",
            ["stroke-width"] = "2",
            ["stroke-linecap"] = "round",
            ["stroke-linejoin"] = "round",
        },
        [
            new SvgShape("rect", new Dictionary<string, string>
            {
                ["x"] = "3", ["y"] = "7", ["width"] = "18", ["height"] = "13", ["rx"] = "2",
            }.ToImmutableDictionary()),
            new SvgShape("path", new Dictionary<string, string>
            {
                ["d"] = "M8 7V5a2 2 0 0 1 2-2h4a2 2 0 0 1 2 2v2",
            }.ToImmutableDictionary()),
        ]));
```

```razor
<sa-icon name="voyager-suitcase"/>
```

**`AddIcon` throws if the name is already taken**, including by a built-in icon. To replace a built-in icon, register a pack.

### An icon pack — `AddIconPack<T>`

For more than a handful of icons, or to override built-in ones, implement `IIconPack` and register it. A pack returns a dictionary of names to definitions; a pack icon whose name is already registered **replaces** the existing one.

```csharp
public class VoyagerIconPack : IIconPack
{
    private static readonly Dictionary<string, string> SvgAttributes = new()
    {
        ["xmlns"] = "http://www.w3.org/2000/svg",
        ["width"] = "24",
        ["height"] = "24",
        ["viewBox"] = "0 0 24 24",
        ["fill"] = "none",
        ["stroke"] = "currentColor",
        ["stroke-width"] = "2",
        ["stroke-linecap"] = "round",
        ["stroke-linejoin"] = "round",
    };

    public IDictionary<string, IconDefinition> GetIcons()
    {
        return new Dictionary<string, IconDefinition>
        {
            ["voyager-compass"] = new IconDefinition(SvgAttributes,
            [
                Shape("circle", ("cx", "12"), ("cy", "12"), ("r", "9")),
                Shape("path", ("d", "m15.5 8.5-2 5-5 2 2-5z")),
            ]),
        };
    }

    private static SvgShape Shape(string name, params (string Name, string Value)[] attributes)
    {
        return new SvgShape(name, attributes.ToImmutableDictionary(a => a.Name, a => a.Value));
    }
}
```

```csharp
builder.Services.AddStellarAdmin()
    .AddTagHelpers()
    .AddIconPack<VoyagerIconPack>();
```

`TIconPack` must have a parameterless constructor. Packs are read once, at registration, and their icons are held in memory for the lifetime of the application.

## API reference

| Member | Description |
|--------|-------------|
| `AddIcon(name, iconDefinition)` | Registers a single icon under a new name. Throws when the name is already registered. |
| `AddIconPack<TIconPack>()` | Instantiates `TIconPack` and registers every icon it returns. Same-named icons replace existing ones. |
| `IIconPack.GetIcons()` | Returns the icons in the pack, keyed by name (`IDictionary<string, IconDefinition>`). |
| `IconDefinition.Attributes` | Attributes rendered on the `<svg>` element (`viewBox`, `fill`, `stroke`, …). Attributes on `<sa-icon>` take precedence. |
| `IconDefinition.Shapes` | The shapes rendered inside the `<svg>`, in order (`List<SvgShape>`). |
| `SvgShape.Name` | The SVG element name — `path`, `circle`, `rect`, `line`, `polyline`. |
| `SvgShape.Attributes` | Attributes on the shape element — `d`, `cx`, `cy`, `r` (`IImmutableDictionary<string, string>`). |

Types live in the `StellarAdmin.TagHelpers.Icons` namespace.
