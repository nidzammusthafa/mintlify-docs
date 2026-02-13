# SPARC Logic: User Flow Documentation

This document outlines the user flows for each feature to be documented.

---

## Feature 1: WhatsApp Blast & Multi-Account

### Flow 1.1: Setup WhatsApp Account
```
1. User navigates to "Accounts" menu
2. Clicks "Add Account" button
3. Chooses connection method:
   a. QR Code (scan with phone)
   b. Pairing Code (enter phone + code)
4. Authenticates WhatsApp account
5. Account appears in list with "Connected" status
6. (Optional) Configure account settings (auto-start, limits)
```

### Flow 1.2: Create Blast Campaign
```
1. User navigates to "Blast" menu
2. Clicks "New Campaign"
3. Enters campaign name
4. Selects sender account(s) (one or multiple)
5. Uploads recipient file (Excel/CSV)
   - System auto-detects phone column
   - User confirms or selects manually
6. Configures message content:
   - Adds text block(s)
   - (Optional) Adds media (image/document/voice)
   - Uses variables for personalization
7. Configures blast settings:
   - Selects speed preset (Safe/Normal/Turbo)
   - Sets custom delays (optional)
8. (Optional) Enables Account Warmer
9. (Optional) Sets schedule with time windows
10. Reviews and clicks "Launch Campaign"
```

### Flow 1.3: Monitor Campaign
```
1. Campaign status changes to "Processing"
2. User sees real-time progress (sent/total)
3. (Optional) Pauses/Resumes campaign
4. On completion, views detailed logs
5. Analyzes delivery success rates
```

---

## Feature 2: Google Maps Scraper

### Flow 2.1: Scrape by Query
```
1. User navigates to "Scraper" > "Maps Scraper"
2. Clicks "New Scraping"
3. Selects "Query Search" method
4. Enters search query (e.g., "Restaurant di Jakarta")
5. Sets location (city/region)
6. Sets result limit
7. Clicks "Start Scraping"
8. Monitors real-time progress
9. Exports results to CSV/Excel
```

### Flow 2.2: Scrape by Coordinates
```
1. User navigates to "Scraper" > "Maps Scraper"
2. Clicks "New Scraping"
3. Selects "Coordinate Search" method
4. Enters latitude and longitude
5. Sets radius (in meters/km)
6. Sets result limit
7. Clicks "Start Scraping"
8. Monitors progress
9. Exports results
```

---

## Feature 3: AI Chatbot & Flow Builder

### Flow 3.1: Setup AI Agent
```
1. User navigates to "AI Settings" or "AI Agent"
2. Selects AI Provider (Gemini default)
3. Enters API Key
4. (Optional) Tests connection
5. Configures AI settings:
   - System prompt (personality)
   - Temperature (creativity)
   - Memory size (conversation context)
   - Max tokens
6. Enables AI Agent
7. Saves configuration
```

### Flow 3.2: Create Simple Auto-Reply
```
1. User navigates to "Auto-Reply" menu
2. Clicks "New Rule"
3. Enters rule name
4. Sets trigger keywords
5. Selects match type (contains/exact/startsWith)
6. Selects response template
7. (Optional) Sets schedule
8. Activates rule
```

### Flow 3.3: Create Flow with Visual Builder
```
1. User navigates to "Flow Builder"
2. Clicks "New Flow"
3. Defines trigger:
   - Keyword trigger (e.g., "halo")
   - Or triggers on all messages
4. Adds nodes by drag-and-drop:
   - Message nodes (text/media)
   - AI Response nodes
   - Condition nodes (if/else)
   - Variable nodes (capture input)
   - HTTP Request nodes
5. Connects nodes by dragging handles
6. Configures each node properties
7. Tests flow in Simulator
8. Activates flow
9. Sets priority (if multiple flows)
```

---

## Feature 4: Account Warmer

### Flow 4.1: Run Account Warmer
```
1. User navigates to "Warmer" menu
2. Clicks "New Warmer Job"
3. Selects account to warm
4. Selects preset or custom delay
5. Selects conversation corpus
6. Sets completion mode (messages/duration/pairs)
7. Clicks "Start Warmer"
8. Monitors warming progress
9. Views completion status
```

---

## Documentation Page Structure

### Template: How-To Guide
```
---
title: "[Feature Name]"
description: "Cara menggunakan [feature]..."
---

## Prerequisites
- List of requirements

## Step 1: [Action]
Description and screenshot

## Step 2: [Action]
Description and screenshot

## Best Practices
- Tips for optimal results

## Troubleshooting
- Common issues and solutions
```
