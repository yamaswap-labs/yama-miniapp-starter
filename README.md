<div align="right">

**English** | [**简体中文**](./README.zh-CN.md)

</div>

---

# 🚀 MiniApp-Starter: Farcaster Mini App Scaffold

Welcome to MiniApp-Starter! This is a simple scaffold based on **Next.js**, helping you quickly configure the **Manifest** and **Embed** for Farcaster Mini Apps. It automates the generation of Manifest files (supporting ngrok for local testing) and, combined with Next.js's standard metadata mechanism, provides a simplified wrapper function for Embed tags, allowing you to easily implement social card previews and interactions.

## 📚 Basic Concepts

Mini App integration hinges on two core pillars: **Manifest** and **Embed**. Each plays a unique role in ensuring your App is discovered, verified, and interactive within the Farcaster ecosystem. Here's a breakdown of their roles, setup, and tips:

### 📄 Manifest (`public/.well-known/farcaster.json`)

The Manifest is your Mini App's **digital ID card** 🪪, declaring metadata to Farcaster clients for automatic domain verification, App icon/name display, and unlocking advanced features like Quick Auth and Actions.

**Key Purpose**: Signals to Farcaster that your domain hosts a Mini App, enabling ecosystem-wide recognition.

### 🎨 Embed (Page Meta Tags)

Embed crafts the **social preview card** for your webpage via HTML meta tags (e.g., `fc:miniapp`), shaping how your page appears in Farcaster posts. Beyond a plain link, it evolves your dApp into a vibrant, clickable social magnet: eye-catching images, titles, descriptions, and buttons that spark instant engagement in the Feed.

**Core Impact**: Enriches dApp cards for richer visuals, operable actions, and magnetic appeal—click to act, flow seamlessly into the Mini App modal for operations.

For full config specs, dive into the [official docs](https://miniapps.farcaster.xyz/docs/specification).

## ⚙️ Manifest Configuration Automation

Farcaster centralizes Mini App identity in `public/.well-known/farcaster.json`. But debugging demands a non-local URL (not `localhost:3000`), so we proxy via `ngrok` for local tests. To avoid manual edits, our **build automation** dynamically generates `farcaster.json`—effortless iteration!

### 🎯 Quick Start Steps

1.**🔌 Launch ngrok Proxy** (for port 3000):

   ```bash
   ngrok http 3000
   ```

2.**📝 Set Domain in `.env.local`**:

   ```bash
   # From ngrok logs:
   # Forwarding    https://3fc5202bb4c2.ngrok-free.app -> http://localhost:3000

   # Add to .env.local (domain only):
   NEXT_PUBLIC_APP_DOMAIN=3fc5202bb4c2.ngrok-free.app
   ```

3 **▶️ Fire Up the Project**:

   ```bash
   pnpm run dev
   ```

4.**📊 Check Terminal Logs**:
   The script outputs all debug essentials (customize via `/scripts/generate-farcaster-json.cjs`). Copy-paste ready for Farcaster tests:

   ![dev-logs](./docs/images/dev-logs.png)

## 🏗️ Page Embed Meta Tag Setup

In Next.js, metadata lives in page-specific `layout.ts` (server components). Think of Farcaster Embed as supercharged HTML metadata: custom tags Farcaster parses for social magic.

**Essence**: A `<meta name="fc:miniapp">` with JSON-stringified config (per official schema)—deserializes to power previews and actions.

**HTML Pseudocode**:

```html
<html>
 <head>
  <!-- Farcaster Embed Magic (Core Meta) -->
  <meta property="fc:miniapp" content='{
    "version": "1",
    "imageUrl": {
      "title": "Join Now!",
      "action": {
        "type": "launch_miniapp",
        "url": "https://your-domain.com/xxx"
      }
    }
  }' />
 </head>
</html>
```

**Tip**: We've wrapped it in `@/lib/generateFrameMetadata.ts` for Next.js ease. Call it in layouts — universal template below:

```ts
// app/xxx/layout.ts

// ... other imports
import { APP_BASE_URL } from '@/lib/constants';
import { generateFrameMetadata } from '@/lib/generateFrameMetadata';
import type { Metadata } from 'next';

export async function generateMetadata(): Promise<Metadata> {
  // Base metadata
  const metadata: Metadata = {
    title: 'Your Page Name',
    description: 'Your Page Description',
  };

  // Farcaster card config (skip for non-social pages)
  const frameUrl = `${APP_BASE_URL}/xxx`;

  // See full fields: https://miniapps.farcaster.xyz/docs/specification
  const frameMetadata = await generateFrameMetadata({
    name: metadata.title as string,
    title: metadata.title as string,
    url: frameUrl,
    description: metadata.description as string,
    imageUrl: `${frameUrl}/opengraph-image`, // Dynamic OG image: https://nextjs.org/docs/app/api-reference/file-conventions/metadata/opengraph-image#generate-images-using-code-js-ts-tsx
    launchButtonName: 'Click to XXX',
  });

  return {
    ...metadata,
    ...frameMetadata,
  };
}

export default function Layout({
  children,
}: Readonly<{
  children: React.ReactNode;
}>) {
  return (
    <html>
      <body>{children}</body>
    </html>
  );
}
```

*(Note: `generateFrameMetadata` starts with `launch_miniapp` action—extend for more!)*

## 🧪 How to Test

After setting the Manifest or page Embed tag, validate with Farcaster's official debugger: [https://farcaster.xyz/~/developers/mini-apps/debug](https://farcaster.xyz/~/developers/mini-apps/debug).

**Tool Interface** (enter your URL for checks):

![developer-tool](./docs/images/developer-tool.png)

- **Mobile Testing**: Farcaster App's developer menu for on-device flow.
- **Real-World Cards**: Cast a test post with your page link—truest preview!

*(As previous mentioned ➡️ Grab test data from logs — easy copy!)*

## 🔗 References

1. **Farcaster Docs**: [https://miniapps.farcaster.xyz/](https://miniapps.farcaster.xyz/)
2. **Ngrok**: [https://ngrok.com/](https://ngrok.com/)
