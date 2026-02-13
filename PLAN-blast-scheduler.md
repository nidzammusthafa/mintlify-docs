# PLAN: Blast/Warmer Scheduler System

**Plan ID:** `blast-scheduler`  
**Created:** 2026-02-02  
**Project Type:** BACKEND + FRONTEND  
**Estimated Duration:** 8-10 hours

---

## Overview

Implement scheduled execution for Blast/Warmer jobs with:

- Multiple time windows per day (e.g., 10:00-12:00, 16:00-17:30)
- Auto-start when entering time window
- Auto-pause when exiting time window
- Countdown display (time until next run/stop)

---

## 📋 Task Breakdown

### Phase 1: Database Schema Updates

#### [MODIFY] `new-server/prisma/schema.prisma`

```prisma
// Add to BlastJob model
model BlastJob {
  // ... existing fields

  // Schedule Configuration
  scheduleConfig    Json?             // ScheduleConfig object
  scheduleType      ScheduleType      @default(IMMEDIATE)
  pauseReason       PauseReason?
  nextWindowStart   DateTime?         // Next time window opens
  nextWindowEnd     DateTime?         // Current window closes

  // ... rest of existing fields
}

enum ScheduleType {
  IMMEDIATE       // Run now (default)
  SCHEDULED       // Waiting for start date/time
  TIME_PAUSED     // Paused - outside time window
}

enum PauseReason {
  USER_REQUESTED   // Manually paused by user
  OUTSIDE_WINDOW   // Auto-paused by scheduler
  DAILY_LIMIT      // Daily message limit reached
  ERROR            // Error occurred
}

// Apply same schema to WarmerJob
model WarmerJob {
  // ... existing fields
  scheduleConfig    Json?
  scheduleType      ScheduleType      @default(IMMEDIATE)
  pauseReason       PauseReason?
  nextWindowStart   DateTime?
  nextWindowEnd     DateTime?
}
```

#### ScheduleConfig Interface

```typescript
// new-server/src/shared/types/schedule.types.ts

export interface ScheduleConfig {
  type: "immediate" | "scheduled";
  timezone: string; // e.g., "Asia/Jakarta"
  startDate: string; // ISO date: "2026-02-03"
  endDate?: string; // Optional end date
  timeWindows: TimeWindow[];
  daysOfWeek?: number[]; // 0-6 (Sun-Sat), null = every day
  behavior: {
    onWindowExit: "finish_current" | "hard_stop";
    dailyLimitReset: "midnight" | "first_window";
  };
}

export interface TimeWindow {
  start: string; // "10:00"
  end: string; // "12:00"
}
```

```bash
# Run migration
cd new-server && npx prisma migrate dev --name add-schedule-config
```

---

### Phase 2: Backend Scheduler Service

#### [NEW] `new-server/src/core/scheduler/scheduler.module.ts`

```typescript
@Module({
  imports: [ScheduleModule.forRoot()],
  providers: [SchedulerService],
  exports: [SchedulerService],
})
export class SchedulerModule {}
```

#### [NEW] `new-server/src/core/scheduler/scheduler.service.ts`

```typescript
@Injectable()
export class SchedulerService {
  private readonly logger = new Logger(SchedulerService.name);

  constructor(
    private prisma: PrismaService,
    private blastService: BlastService,
    private warmerService: WarmerService,
    private gateway: WhatsappGateway,
  ) {}

  @Cron("* * * * *") // Every minute
  async checkScheduledJobs() {
    const now = new Date();

    // 1. Start SCHEDULED jobs that should begin
    await this.startScheduledJobs(now);

    // 2. Pause RUNNING jobs that exit time window
    await this.pauseOutOfWindowJobs(now);

    // 3. Resume TIME_PAUSED jobs entering time window
    await this.resumePausedJobs(now);

    // 4. Update next window times for all scheduled jobs
    await this.updateNextWindowTimes(now);
  }

  isWithinTimeWindow(config: ScheduleConfig, now: Date): boolean {
    const tz = config.timezone || "Asia/Jakarta";
    const currentTime = formatInTimeZone(now, tz, "HH:mm");
    const currentDay = getDay(toZonedTime(now, tz));

    // Check day of week
    if (config.daysOfWeek && !config.daysOfWeek.includes(currentDay)) {
      return false;
    }

    // Check time windows
    return config.timeWindows.some(
      (w) => currentTime >= w.start && currentTime < w.end,
    );
  }

  calculateNextWindow(
    config: ScheduleConfig,
    now: Date,
  ): {
    nextStart: Date | null;
    nextEnd: Date | null;
  } {
    // Calculate next window start and current window end
    // ... implementation
  }
}
```

#### [MODIFY] `new-server/src/modules/blast/blast.service.ts`

```typescript
// Update createBlastJob to handle scheduling
async createBlastJob(dto: CreateBlastJobDto) {
  const scheduleConfig = dto.scheduleConfig;

  let status: BlastJobStatus;
  let scheduleType: ScheduleType;

  if (scheduleConfig?.type === 'scheduled') {
    status = 'SCHEDULED';
    scheduleType = 'SCHEDULED';
  } else {
    status = 'PENDING';
    scheduleType = 'IMMEDIATE';
  }

  const job = await this.prisma.blastJob.create({
    data: {
      ...dto,
      status,
      scheduleType,
      scheduleConfig: scheduleConfig as any,
      nextWindowStart: this.calculateNextWindowStart(scheduleConfig),
      nextWindowEnd: this.calculateNextWindowEnd(scheduleConfig),
    },
  });

  return job;
}
```

#### API Response Enhancement

```typescript
// Add to BlastJob response
interface BlastJobResponse {
  // ... existing fields
  scheduleInfo: {
    type: "immediate" | "scheduled";
    status: "waiting" | "running" | "paused" | "completed";
    nextWindowStart: string | null;
    nextWindowEnd: string | null;
    countdown: {
      untilStart: number | null; // seconds until next run
      untilEnd: number | null; // seconds until pause
      formatted: string; // "2h 15m" or "3d 4h"
    };
    timeWindows: TimeWindow[];
    daysOfWeek: number[];
  };
}
```

---

### Phase 3: Frontend - Schedule Form Component

#### [NEW] `new-client/src/components/scheduling/schedule-form.tsx`

**Design Specifications (ui-ux-pro-max):**

- Use shadcn components
- Border radius: 0.5rem
- Smooth transitions: 200ms
- Clear visual hierarchy

```tsx
"use client";

import { useState } from "react";
import { Calendar } from "@/components/ui/calendar";
import { Switch } from "@/components/ui/switch";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Checkbox } from "@/components/ui/checkbox";
import { Label } from "@/components/ui/label";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Plus, Trash2, Clock, Calendar as CalendarIcon } from "lucide-react";

interface ScheduleFormProps {
  value: ScheduleConfig | null;
  onChange: (config: ScheduleConfig | null) => void;
}

export function ScheduleForm({ value, onChange }: ScheduleFormProps) {
  const [isScheduled, setIsScheduled] = useState(value?.type === 'scheduled');
  const [timeWindows, setTimeWindows] = useState<TimeWindow[]>(
    value?.timeWindows || [{ start: "10:00", end: "12:00" }]
  );
  const [daysOfWeek, setDaysOfWeek] = useState<number[]>(
    value?.daysOfWeek || [1, 2, 3, 4, 5]
  );

  const addTimeWindow = () => {
    setTimeWindows([...timeWindows, { start: "16:00", end: "17:30" }]);
  };

  const removeTimeWindow = (index: number) => {
    setTimeWindows(timeWindows.filter((_, i) => i !== index));
  };

  return (
    <Card className="border border-border/50 bg-card/50 backdrop-blur-sm">
      <CardHeader className="pb-4">
        <div className="flex items-center justify-between">
          <CardTitle className="text-lg flex items-center gap-2">
            <Clock className="h-5 w-5 text-primary" />
            Schedule
          </CardTitle>
          <Switch
            checked={isScheduled}
            onCheckedChange={(v) => {
              setIsScheduled(v);
              onChange(v ? buildConfig() : null);
            }}
          />
        </div>
      </CardHeader>

      {isScheduled && (
        <CardContent className="space-y-6">
          {/* Date Selection */}
          <div className="grid grid-cols-2 gap-4">
            <div className="space-y-2">
              <Label>Start Date</Label>
              <Calendar mode="single" ... />
            </div>
            <div className="space-y-2">
              <Label>End Date (Optional)</Label>
              <Calendar mode="single" ... />
            </div>
          </div>

          {/* Time Windows */}
          <div className="space-y-3">
            <Label className="flex items-center gap-2">
              <Clock className="h-4 w-4" />
              Time Windows
            </Label>

            {timeWindows.map((window, idx) => (
              <div key={idx} className="flex items-center gap-2">
                <Input
                  type="time"
                  value={window.start}
                  onChange={(e) => updateWindow(idx, 'start', e.target.value)}
                  className="w-28"
                />
                <span className="text-muted-foreground">–</span>
                <Input
                  type="time"
                  value={window.end}
                  onChange={(e) => updateWindow(idx, 'end', e.target.value)}
                  className="w-28"
                />
                {timeWindows.length > 1 && (
                  <Button
                    variant="ghost"
                    size="icon"
                    onClick={() => removeTimeWindow(idx)}
                    className="text-destructive hover:text-destructive/80"
                  >
                    <Trash2 className="h-4 w-4" />
                  </Button>
                )}
              </div>
            ))}

            <Button
              variant="outline"
              size="sm"
              onClick={addTimeWindow}
              className="w-full mt-2"
            >
              <Plus className="h-4 w-4 mr-2" />
              Add Time Window
            </Button>
          </div>

          {/* Days of Week */}
          <div className="space-y-3">
            <Label>Active Days</Label>
            <div className="flex gap-2 flex-wrap">
              {['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat'].map((day, idx) => (
                <DayToggle
                  key={day}
                  day={day}
                  checked={daysOfWeek.includes(idx)}
                  onCheckedChange={(checked) => toggleDay(idx, checked)}
                />
              ))}
            </div>
          </div>
        </CardContent>
      )}
    </Card>
  );
}

function DayToggle({ day, checked, onCheckedChange }) {
  return (
    <button
      onClick={() => onCheckedChange(!checked)}
      className={cn(
        "w-10 h-10 rounded-full text-sm font-medium transition-colors",
        "border cursor-pointer",
        checked
          ? "bg-primary text-primary-foreground border-primary"
          : "bg-muted text-muted-foreground border-border hover:bg-muted/80"
      )}
    >
      {day[0]}
    </button>
  );
}
```

---

### Phase 4: Countdown Display Component

#### [NEW] `new-client/src/components/scheduling/schedule-countdown.tsx`

```tsx
"use client";

import { useEffect, useState } from "react";
import { formatDistanceToNow, differenceInSeconds } from "date-fns";
import { id } from "date-fns/locale";
import { Clock, Play, Pause, Timer } from "lucide-react";
import { cn } from "@/lib/utils";

interface ScheduleCountdownProps {
  nextWindowStart: string | null;
  nextWindowEnd: string | null;
  status: "waiting" | "running" | "paused" | "completed";
}

export function ScheduleCountdown({
  nextWindowStart,
  nextWindowEnd,
  status,
}: ScheduleCountdownProps) {
  const [now, setNow] = useState(new Date());

  useEffect(() => {
    const interval = setInterval(() => setNow(new Date()), 1000);
    return () => clearInterval(interval);
  }, []);

  const getCountdown = (target: string | null) => {
    if (!target) return null;
    const targetDate = new Date(target);
    const seconds = differenceInSeconds(targetDate, now);
    if (seconds <= 0) return null;
    return formatDuration(seconds);
  };

  const startCountdown = getCountdown(nextWindowStart);
  const endCountdown = getCountdown(nextWindowEnd);

  return (
    <div className="flex items-center gap-4">
      {/* Status Badge */}
      <StatusBadge status={status} />

      {/* Countdown Info */}
      {status === "waiting" && startCountdown && (
        <CountdownPill
          icon={<Play className="h-3.5 w-3.5" />}
          label="Starts in"
          value={startCountdown}
          variant="success"
        />
      )}

      {status === "running" && endCountdown && (
        <CountdownPill
          icon={<Pause className="h-3.5 w-3.5" />}
          label="Pauses in"
          value={endCountdown}
          variant="warning"
        />
      )}

      {status === "paused" && startCountdown && (
        <CountdownPill
          icon={<Timer className="h-3.5 w-3.5" />}
          label="Resumes in"
          value={startCountdown}
          variant="info"
        />
      )}
    </div>
  );
}

function StatusBadge({ status }: { status: string }) {
  const config = {
    waiting: {
      label: "Scheduled",
      color: "bg-blue-500/10 text-blue-500 border-blue-500/20",
    },
    running: {
      label: "Running",
      color: "bg-emerald-500/10 text-emerald-500 border-emerald-500/20",
    },
    paused: {
      label: "Paused",
      color: "bg-amber-500/10 text-amber-500 border-amber-500/20",
    },
    completed: {
      label: "Completed",
      color: "bg-slate-500/10 text-slate-500 border-slate-500/20",
    },
  }[status];

  return (
    <span
      className={cn(
        "px-2.5 py-1 text-xs font-medium rounded-full border",
        config.color,
      )}
    >
      {config.label}
    </span>
  );
}

function CountdownPill({ icon, label, value, variant }) {
  const colors = {
    success: "bg-emerald-500/10 text-emerald-600 dark:text-emerald-400",
    warning: "bg-amber-500/10 text-amber-600 dark:text-amber-400",
    info: "bg-blue-500/10 text-blue-600 dark:text-blue-400",
  };

  return (
    <div
      className={cn(
        "flex items-center gap-2 px-3 py-1.5 rounded-lg text-sm",
        colors[variant],
      )}
    >
      {icon}
      <span className="text-muted-foreground">{label}</span>
      <span className="font-mono font-semibold">{value}</span>
    </div>
  );
}

function formatDuration(seconds: number): string {
  const days = Math.floor(seconds / 86400);
  const hours = Math.floor((seconds % 86400) / 3600);
  const minutes = Math.floor((seconds % 3600) / 60);
  const secs = seconds % 60;

  if (days > 0) return `${days}d ${hours}h`;
  if (hours > 0) return `${hours}h ${minutes}m`;
  if (minutes > 0) return `${minutes}m ${secs}s`;
  return `${secs}s`;
}
```

---

### Phase 5: Integration with Blast/Warmer Pages

#### [MODIFY] `new-client/src/app/(dashboard)/blast/page.tsx`

```tsx
// Add ScheduleForm to blast job creation
import { ScheduleForm } from "@/components/scheduling/schedule-form";
import { ScheduleCountdown } from "@/components/scheduling/schedule-countdown";

// In job list, show countdown for each scheduled job
{
  jobs.map((job) => (
    <JobCard key={job.id}>
      <JobHeader>
        <JobTitle>{job.name}</JobTitle>
        {job.scheduleConfig && (
          <ScheduleCountdown
            nextWindowStart={job.nextWindowStart}
            nextWindowEnd={job.nextWindowEnd}
            status={getScheduleStatus(job)}
          />
        )}
      </JobHeader>
      {/* ... rest of card */}
    </JobCard>
  ));
}
```

#### [MODIFY] `new-client/src/app/(dashboard)/warmer/page.tsx`

Apply same pattern as blast page.

---

### Phase 6: WebSocket Events

#### [MODIFY] `new-server/src/gateway/whatsapp.gateway.ts`

```typescript
// Add schedule-related events
@WebSocketGateway({ namespace: "/whatsapp" })
export class WhatsappGateway {
  // Emit schedule status changes
  emitScheduleUpdate(
    jobId: string,
    type: "blast" | "warmer",
    data: ScheduleUpdate,
  ) {
    this.server.emit("schedule:update", {
      jobId,
      type,
      status: data.status,
      nextWindowStart: data.nextWindowStart,
      nextWindowEnd: data.nextWindowEnd,
      countdownSeconds: data.countdownSeconds,
    });
  }
}
```

#### Frontend WebSocket Hook

```typescript
// new-client/src/hooks/use-schedule-updates.ts
export function useScheduleUpdates(jobId: string) {
  const [scheduleInfo, setScheduleInfo] = useState<ScheduleInfo | null>(null);

  useSocket("schedule:update", (data) => {
    if (data.jobId === jobId) {
      setScheduleInfo(data);
    }
  });

  return scheduleInfo;
}
```

---

## 🎨 UI Design Specifications

### Color Palette (from design tokens)

| State             | Background       | Text          | Border           |
| ----------------- | ---------------- | ------------- | ---------------- |
| Waiting/Scheduled | `blue-500/10`    | `blue-500`    | `blue-500/20`    |
| Running           | `emerald-500/10` | `emerald-500` | `emerald-500/20` |
| Paused            | `amber-500/10`   | `amber-500`   | `amber-500/20`   |
| Completed         | `slate-500/10`   | `slate-500`   | `slate-500/20`   |
| Error             | `red-500/10`     | `red-500`     | `red-500/20`     |

### Components

- **DayToggle:** Circular buttons (40px), selected = primary color
- **TimeWindow Input:** Side-by-side time inputs with dash separator
- **Countdown Pill:** Rounded-lg, icon + label + monospace value
- **Status Badge:** Rounded-full, subtle background

### Animations

```css
/* Countdown pulse when < 1 minute */
@keyframes countdown-pulse {
  0%,
  100% {
    opacity: 1;
  }
  50% {
    opacity: 0.7;
  }
}

.countdown-urgent {
  animation: countdown-pulse 1s ease-in-out infinite;
}
```

---

## 🧪 Verification Plan

### Backend Tests

1. Create blast job with schedule → Status = SCHEDULED
2. Wait for time window start → Status = RUNNING
3. Wait for time window end → Status = TIME_PAUSED
4. Next day same time → Status = RUNNING again
5. End date passed → Status = COMPLETED

### Frontend Tests

1. Toggle schedule switch → Form appears/hides
2. Add/remove time windows → Updates correctly
3. Select days of week → Visual toggle works
4. Countdown displays → Updates every second
5. WebSocket update → UI reflects new status

### Integration Tests

```bash
# Test with specific times
npm run test:e2e -- --grep "scheduler"
```

---

## 📁 Files to Create/Modify

### New Files

| File                                                          | Purpose               |
| ------------------------------------------------------------- | --------------------- |
| `new-server/src/shared/types/schedule.types.ts`               | TypeScript interfaces |
| `new-server/src/core/scheduler/scheduler.module.ts`           | NestJS module         |
| `new-server/src/core/scheduler/scheduler.service.ts`          | Cron-based scheduler  |
| `new-client/src/components/scheduling/schedule-form.tsx`      | Schedule config form  |
| `new-client/src/components/scheduling/schedule-countdown.tsx` | Countdown display     |
| `new-client/src/hooks/use-schedule-updates.ts`                | WebSocket hook        |

### Modified Files

| File                                               | Changes                                              |
| -------------------------------------------------- | ---------------------------------------------------- |
| `new-server/prisma/schema.prisma`                  | Add ScheduleType, PauseReason enums, schedule fields |
| `new-server/src/modules/blast/blast.service.ts`    | Handle scheduled job creation                        |
| `new-server/src/modules/blast/blast.controller.ts` | Add schedule fields to DTO                           |
| `new-server/src/modules/warmer/warmer.service.ts`  | Same as blast                                        |
| `new-server/src/gateway/whatsapp.gateway.ts`       | Schedule update events                               |
| `new-client/src/app/(dashboard)/blast/page.tsx`    | Integrate schedule form                              |
| `new-client/src/app/(dashboard)/warmer/page.tsx`   | Integrate schedule form                              |

---

## ⏱️ Execution Order

```
Phase 1 (Schema) → Phase 2 (Backend) → Phase 3 (Form UI)
                                            ↓
                     Phase 6 (WebSocket) ← Phase 4 (Countdown)
                                            ↓
                                     Phase 5 (Integration)
```

**Total Estimated Time:** 8-10 hours
