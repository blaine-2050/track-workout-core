# Business Goals

## North Star
Build an innovative workout tracking product that can iterate rapidly across platforms while preserving trustworthy workout history.

## Goal 1: Fast Experiment Loop
- Use automated computer-use testing to iterate on workflow without manual regression cycles.
- Validate features on the fastest-to-build platform (iOS Simulator) before touching slower ones.

## Goal 2: Reliable Cross-Device Logging
- Users can always log workouts, online or offline.
- Synchronize data to a central repository without losing records.

## Goal 3: Multi-Platform Delivery Without Chaos
- One prose spec (this repo) governs all platforms.
- Each platform is its own repo; none is a monorepo member.
- API and BLE integration stay as separate, versioned concerns.

## Goal 4: Data Foundation for Insights
- Capture consistent event timestamps and metadata for future analytics.
- Enable future heart-rate alignment with workout events.

## Constraints
- Small team, high iteration pressure.
- Physical device testing happens in a real gym; cannot be fully automated.
- Deferring paid Apple Developer account ($99) until TestFlight is needed.
