# Frontend Framework Kit — React/Next.js + PAPI

## Stack

| Layer | Choice |
|-------|--------|
| Framework | Next.js (App Router) |
| Chain client | PAPI (`polkadot-api`) |
| Light client | Smoldot (via PAPI) |
| Wallet connect | PAPI PJS signer / WalletConnect |
| Styling | Your choice (Tailwind, CSS modules, etc.) |

## Project Setup

```bash
# Create Next.js app
npx create-next-app@latest my-polkadot-app --typescript --tailwind --app

cd my-polkadot-app

# Install PAPI
npm install polkadot-api

# Generate chain descriptors
npx papi add dot -n polkadot
npx papi add ah -n polkadot_asset_hub

# Commit generated descriptors
git add .papi/
```

## Chain Provider Pattern

Create a provider that initializes the client once and shares it across the app.

```typescript
// providers/polkadot-provider.tsx
"use client"

import { createClient, TypedApi } from "polkadot-api"
import { getSmProvider } from "polkadot-api/sm-provider"
import { startFromWorker } from "polkadot-api/smoldot/from-worker"
import { chainSpec } from "polkadot-api/chains/polkadot"
import { dot } from "@polkadot-api/descriptors"
import { createContext, useContext, useEffect, useState, ReactNode } from "react"

type PolkadotContextType = {
  api: TypedApi<typeof dot> | null
  isReady: boolean
}

const PolkadotContext = createContext<PolkadotContextType>({
  api: null,
  isReady: false,
})

export function PolkadotProvider({ children }: { children: ReactNode }) {
  const [api, setApi] = useState<TypedApi<typeof dot> | null>(null)
  const [isReady, setIsReady] = useState(false)

  useEffect(() => {
    let cancelled = false

    async function init() {
      const smoldot = startFromWorker(
        new Worker(new URL("polkadot-api/smoldot/worker", import.meta.url))
      )
      const chain = await smoldot.addChain({ chainSpec })
      const client = createClient(getSmProvider(chain))
      const typedApi = client.getTypedApi(dot)

      if (!cancelled) {
        setApi(typedApi)
        setIsReady(true)
      }
    }

    init()
    return () => { cancelled = true }
  }, [])

  return (
    <PolkadotContext.Provider value={{ api, isReady }}>
      {children}
    </PolkadotContext.Provider>
  )
}

export function usePolkadot() {
  return useContext(PolkadotContext)
}
```

## Wallet Connection

```typescript
// hooks/useWallet.ts
"use client"

import { connectInjectedExtension } from "polkadot-api/pjs-signer"
import { useState, useCallback } from "react"
import type { InjectedPolkadotAccount } from "polkadot-api/pjs-signer"

export function useWallet() {
  const [accounts, setAccounts] = useState<InjectedPolkadotAccount[]>([])
  const [selectedAccount, setSelectedAccount] = useState<InjectedPolkadotAccount | null>(null)

  const connect = useCallback(async () => {
    const extension = await connectInjectedExtension("polkadot-js")
    const accts = extension.getAccounts()
    setAccounts(accts)
    if (accts.length > 0) setSelectedAccount(accts[0])
  }, [])

  return {
    accounts,
    selectedAccount,
    setSelectedAccount,
    connect,
    signer: selectedAccount?.polkadotSigner ?? null,
  }
}
```

## Reading Chain State

```typescript
// hooks/useBalance.ts
"use client"

import { usePolkadot } from "@/providers/polkadot-provider"
import { useState, useEffect } from "react"
import type { SS58String } from "polkadot-api"

export function useBalance(address: SS58String | null) {
  const { api, isReady } = usePolkadot()
  const [balance, setBalance] = useState<bigint>(0n)

  useEffect(() => {
    if (!api || !isReady || !address) return

    const subscription = api.query.System.Account.watchValue(address).subscribe((account) => {
      setBalance(account.data.free)
    })

    return () => { subscription.unsubscribe() }
  }, [api, isReady, address])

  return balance
}
```

## Submitting Transactions

```typescript
// hooks/useTransfer.ts
"use client"

import { usePolkadot } from "@/providers/polkadot-provider"
import { useWallet } from "@/hooks/useWallet"
import { MultiAddress } from "@polkadot-api/descriptors"
import { useState } from "react"

export function useTransfer() {
  const { api } = usePolkadot()
  const { signer } = useWallet()
  const [status, setStatus] = useState<string>("idle")

  async function transfer(to: string, amount: bigint) {
    if (!api || !signer) return

    setStatus("signing")

    const tx = api.tx.Balances.transfer_keep_alive({
      dest: MultiAddress.Id(to),
      value: amount,
    })

    try {
      tx.signSubmitAndWatch(signer).subscribe({
        next: (event) => {
          setStatus(event.type)
          if (event.type === "finalized") {
            setStatus("confirmed")
          }
        },
        error: (err) => setStatus(`error: ${err.message}`),
      })
    } catch (err) {
      setStatus("failed")
    }
  }

  return { transfer, status }
}
```

## Display Patterns

### Format DOT Balances

```typescript
// utils/format.ts
export function formatDOT(planck: bigint): string {
  const dot = Number(planck) / 1e10
  return new Intl.NumberFormat("en-US", {
    minimumFractionDigits: 2,
    maximumFractionDigits: 4,
  }).format(dot)
}

// DOT has 10 decimal places
// 1 DOT = 10_000_000_000 planck
export function dotToPlanck(dot: number): bigint {
  return BigInt(Math.round(dot * 1e10))
}
```

### Address Display

```typescript
export function shortAddress(address: string): string {
  return `${address.slice(0, 6)}...${address.slice(-6)}`
}
```

## App Layout

```typescript
// app/layout.tsx
import { PolkadotProvider } from "@/providers/polkadot-provider"

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>
        <PolkadotProvider>
          {children}
        </PolkadotProvider>
      </body>
    </html>
  )
}
```

## Best Practices

1. **Use Smoldot in production** — Light client = no centralized RPC dependency
2. **WebSocket for dev only** — `getWsProvider` is faster to connect but centralized
3. **Watch, don't poll** — Use `watchValue` for reactive UI updates
4. **BigInt for all balances** — Never convert to `number` for arithmetic; only for display
5. **Handle loading states** — Light client sync takes a few seconds; show a loading indicator
6. **Commit `.papi/descriptors/`** — These are generated types; version them like any other code
7. **SSR considerations** — PAPI client initialization uses Web Workers; wrap in `"use client"` components
