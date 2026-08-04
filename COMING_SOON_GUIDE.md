# Coming Soon Popups Implementation Guide

## Overview
This guide shows how to add "Coming Soon" popups to unavailable features in the FraudShield Detection System.

## Components Created

### 1. **ComingSoonDialog** (`frontend/components/ui/coming-soon-dialog.tsx`)
The main popup component that displays the "Coming Soon" message.

**Props:**
- `open` - Boolean to control dialog visibility
- `onOpenChange` - Callback when dialog open state changes
- `feature` - Name of the coming soon feature
- `description` - Optional description of what's coming
- `estimatedDate` - Optional estimated availability date

### 2. **Hooks** (`frontend/hooks/use-coming-soon.ts`)

#### `useComingSoon(feature)`
Simple hook for a single feature.
```typescript
const feature = useComingSoon({
  name: "Advanced Analytics",
  description: "Detailed metrics and trends",
  estimatedDate: "Q3 2024"
})

return (
  <>
    <Button onClick={feature.showDialog}>Analytics</Button>
    <ComingSoonDialog
      open={feature.open}
      onOpenChange={feature.setOpen}
      feature={feature.featureName}
      description={feature.featureDescription}
      estimatedDate={feature.estimatedDate}
    />
  </>
)
```

#### `useComingSoonFeatures()`
Hook for managing multiple "Coming Soon" features in one component.
```typescript
const { showComingSoon, getComingSoonProps } = useComingSoonFeatures()

// In JSX
<Button onClick={() => showComingSoon('export')}>Export Data</Button>
<ComingSoonDialog {...getComingSoonProps('export')} feature="Data Export" />
```

### 3. **ComingSoonButton Component** (`frontend/components/coming-soon-button.tsx`)
Wrapper component that automatically shows the dialog when clicked.

#### Usage 1: Wrapping an existing component
```typescript
<ComingSoonButton 
  feature="API Integration" 
  description="Connect your own integrations"
  estimatedDate="Q2 2024"
>
  <Button>API Settings</Button>
</ComingSoonButton>
```

#### Usage 2: Using the hook
```typescript
const { handleClick, DialogNode } = useComingSoonClickHandler(
  "Advanced Reporting",
  "Generate custom reports",
  "Q3 2024"
)

return (
  <>
    <Button onClick={handleClick}>Reports</Button>
    {DialogNode}
  </>
)
```

## Implementation Examples

### Example 1: Disable Settings Tab Features
**File:** `frontend/components/dashboard/tabs/settings-tab.tsx`

```typescript
import { useComingSoonFeatures } from "@/hooks/use-coming-soon"
import { ComingSoonDialog } from "@/components/ui/coming-soon-dialog"

export function SettingsTab() {
  const { showComingSoon, getComingSoonProps } = useComingSoonFeatures()

  return (
    <>
      {/* Existing content */}
      <Button onClick={() => showComingSoon('advanced-alerts')}>
        Advanced Alert Settings
      </Button>

      {/* Coming Soon Dialogs */}
      <ComingSoonDialog
        {...getComingSoonProps('advanced-alerts')}
        feature="Advanced Alert Settings"
        description="Configure complex alert rules and conditions"
        estimatedDate="Q2 2024"
      />
    </>
  )
}
```

### Example 2: Export Feature
```typescript
import { ComingSoonButton } from "@/components/coming-soon-button"
import { Button } from "@/components/ui/button"

<ComingSoonButton
  feature="Export Transactions"
  description="Download transaction data in multiple formats (CSV, Excel, PDF)"
  estimatedDate="Q1 2024"
>
  <Button variant="outline">
    <Download className="h-4 w-4 mr-2" />
    Export
  </Button>
</ComingSoonButton>
```

### Example 3: Multiple Buttons in One Component
```typescript
import { useComingSoonFeatures } from "@/hooks/use-coming-soon"
import { ComingSoonDialog } from "@/components/ui/coming-soon-dialog"

export function ActionsBar() {
  const { showComingSoon, getComingSoonProps } = useComingSoonFeatures()

  const comingSoonFeatures = {
    export: {
      feature: "Export Data",
      description: "Download data in multiple formats",
      estimatedDate: "Q1 2024"
    },
    schedule: {
      feature: "Schedule Reports",
      description: "Automatically generate and send reports",
      estimatedDate: "Q2 2024"
    },
    api: {
      feature: "API Access",
      description: "Programmatic access to system data",
      estimatedDate: "Q3 2024"
    }
  }

  return (
    <>
      <Button onClick={() => showComingSoon('export')}>Export</Button>
      <Button onClick={() => showComingSoon('schedule')}>Schedule</Button>
      <Button onClick={() => showComingSoon('api')}>API Docs</Button>

      {Object.entries(comingSoonFeatures).map(([key, props]) => (
        <ComingSoonDialog
          key={key}
          {...getComingSoonProps(key)}
          {...props}
        />
      ))}
    </>
  )
}
```

## Features to Consider Marking as "Coming Soon"

Based on the current implementation, consider these features:

1. **Fraud Chains Tab** - Advanced visualization
2. **Risk Scoring** - Advanced ML model configuration
3. **Settings Tab** - API keys, webhooks, advanced configurations
4. **User Profiles** - Bulk operations, advanced filtering
5. **Data Export** - Multiple format exports
6. **Scheduled Reports** - Automatic report generation
7. **Integrations** - Third-party service connections
8. **Advanced Analytics** - Custom metric creation

## Quick Implementation Checklist

- [ ] Review which features/buttons should show "Coming Soon"
- [ ] Import the appropriate component/hook (ComingSoonDialog, ComingSoonButton, or useComingSoon)
- [ ] Add onClick handler that shows the dialog
- [ ] Customize the feature name, description, and estimated date
- [ ] Test the dialog appears and can be dismissed
- [ ] (Optional) Wire "Notify me" button to email/notification system

## Styling

The Coming Soon dialog automatically uses your app's theming:
- Primary color for the icon background
- Foreground/muted colors for text hierarchy
- Respects dark/light mode

No additional styling needed!
