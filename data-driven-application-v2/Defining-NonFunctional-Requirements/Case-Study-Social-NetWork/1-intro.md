# Social Network System Design
### A Twitter-style microblogging platform

---

## Problem Statement

Design a social network in the style of Twitter where users can:
- **Post messages** to share with their followers
- **Follow other users** to see their posts in a feed

---

## Scale & Assumptions

### Traffic
| Metric | Value |
|--------|-------|
| Posts per day | 500 million |
| Average posts per second | 5,800 |
| Peak posts per second | 150,000 |

### User Behaviour
| Metric | Value |
|--------|-------|
| Avg. accounts followed per user | 200 |
| Avg. followers per user | 200 |
