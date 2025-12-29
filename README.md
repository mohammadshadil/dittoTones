# dittoTones 🟣

A mini-library to transform any color into a full palette, based on the perceptual "DNA" of any design system.

Demo: https://meodai.github.io/dittoTones/

## How it works

Most palette generators for popular frameworks either match a single color or ignore the careful work that was put into creating the original palettes entirely. dittoTones takes a different approach: it analyzes the perceptual "DNA" (Lightness and Chroma curves in Oklch space) of popular design systems like Tailwind or Radix. It then maps your target hue onto these curves, ensuring your custom palette maintains similar accessible contrast ratios and vibrancy as the reference system.

```text
       Input Color
           │
           ▼
   1. Find Neighbors
      (in Oklch)
           │
    ┌──────┴──────┐
    ▼             ▼
 Ramp A        Ramp B
 (Blue)        (Cyan)
    │             │
    ▼             ▼
 L/C Curve     L/C Curve
    │             │
    └──────┬──────┘
           │
           ▼
   2. Blend Curves
      (Weighted)
           │
           ▼
    3. Apply Hue
           │
           ▼
   Generated Palette
```

## Install

```bash
npm install dittotones
```

## Usage

```typescript
import { DittoTones } from 'dittotones';
import { tailwindRamps } from 'dittotones/ramps/tailwind';
// or
import { radixRamps } from 'dittotones/ramps/radix';
import { formatCss, formatHex } from 'culori';

// Use Tailwind ramps (shades: 50-950)
const ditto = new DittoTones({ ramps: tailwindRamps });

// Or Radix ramps (shades: 1-12)
const dittoRadix = new DittoTones({ ramps: radixRamps });

const result = ditto.generate('#F97316');

// result.scale contains Oklch color objects
// Use culori's formatCss or formatHex to convert:

for (const [shade, color] of Object.entries(result.scale)) {
  console.log(`${shade}: ${formatCss(color)}`);
  // 50: oklch(0.98 0.016 49)
  // 100: oklch(0.954 0.038 49)
  // ...
}

// Or as hex:
for (const [shade, color] of Object.entries(result.scale)) {
  console.log(`${shade}: ${formatHex(color)}`);
}
```

## Result

```typescript
interface GenerateResult {
  inputColor: Oklch; // Parsed input color
  matchedShade: string; // e.g. "500"
  method: 'exact' | 'single' | 'blend';
  sources: {
    // Which ramps were used
    name: string;
    diff: number;
    weight: number;
  }[];
  scale: Record<string, Oklch>; // The generated palette
}
```

## How it works

1. **Parse input** — converts the input into `Oklch` via `culori`
2. **Handle neutrals** — if chroma is very low, picks the “most neutral” ramp and returns it as-is
3. **Find closest match** — finds the nearest ramp color by Euclidean distance in OKLCH (`diff`)
4. **Pick strategy** — `exact` if `diff` is below a small threshold, otherwise `single` (one ramp) or `blend` (two ramps; second ramp chosen by closest hue at the matched shade)
5. **Rotate hue + correct L/C** — sets the target hue across the scale, then offsets lightness and scales chroma so the matched shade lands on the input color

## Custom ramps

```typescript
import { DittoTones } from 'dittotones';
import { parse, oklch, type Oklch } from 'culori';

const customRamps = new Map([
  [
    'brand',
    {
      '50': oklch(parse('oklch(98% 0.01 250)')) as Oklch,
      '500': oklch(parse('#3B82F6')) as Oklch,
      '950': oklch(parse('oklch(25% 0.05 250)')) as Oklch,
    },
  ],
]);

const ditto = new DittoTones({ ramps: customRamps });
```

## Dev

```bash
npm install
npm run dev     # Start dev server with demo
npm run build   # Build library
npm run preview # Preview the demo build
```

## Notes

- ESM-only package (`"type": "module"`).

## Credits

Built with [Culori](https://culorijs.org/) for color math and interpolation.

## License

MIT
