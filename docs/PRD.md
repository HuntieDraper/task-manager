# Product Requirements Document: Task Manager

| | |
|---|---|
| **Status** | Draft, in review |
| **Author** | Hunter |
| **Last updated** | 7 Nov 2026 |
| **Related docs** | [PROGRESS.md](PROGRESS.md), Technical design (to come), ADRs (to come) |

---

## 1. Problem statement

I currently track my tasks and chores on paper. Paper works for one-off tasks, but it doesn't handle **recurring** tasks: every daily task has to be rewritten every day, and chores on a schedule (e.g. every Wednesday and Saturday, or once in the first week of the month) have to be remembered and copied over by hand. That rewriting is tedious, so I stopped keeping the list, and without it I lose track of what needs doing and lose motivation.

I need a tool I can check on my phone throughout the day that **knows my schedules for me** and shows only what matters right now: what's overdue, what's due today, and what's still left to hit this week.

## 2. Goals

1. **Never rewrite a recurring task.** You set up a task and its schedule once, and the app works out when it's due.
2. **Show what matters now.** Opening the app shows a single prioritised list of today's important tasks, with overdue items first.
3. **Keep an honest record.** The app records which occurrences were completed and which were missed, so I can look back at any past day.
4. **Usable on a phone.** The main way of using the app is checking it in a mobile browser throughout the day.

### Success metrics

| Metric | Target |
|---|---|
| Real-world use | I use the app instead of paper as my only task tracker for **7 consecutive days**. |
| Coverage | All of my example tasks (see Appendix A) can be set up in the app with the correct schedule. |
| Quality | Recurrence, overdue and sorting logic is covered by automated tests that run in CI on every pull request. |

### Non-goals

- Replacing a full calendar app or project management tool.
- Collaboration or sharing tasks between people.
- Motivational features such as streaks, stats or gamification.

## 3. User personas

### Primary: "Busy student" (me)
- 2nd-year software engineering student whose week changes a lot between uni, work and home.
- Has a mix of one-off deadlines (assignments), daily habits (creatine, skincare) and household chores on set schedules (pet care, cleaning).
- Mostly on a **phone**, glancing at the list several times a day.
- **Pain:** paper lists don't repeat, so recurring tasks get forgotten and the list gets abandoned.
- **Needs:** set up once, see "what do I need to do now" at a glance, tick things off quickly.

### Secondary: "Another individual user"
- Anyone else who signs up for their own account (friend, housemate).
- Uses the app the same way as the primary persona, with their **own private tasks**.
- Does not share lists with anyone.

## 4. Key concepts and definitions

These terms are used throughout the user stories, so they're defined once here.

| Term | Definition |
|---|---|
| **Task** | Something the user wants to do, with a name, a schedule, a priority and optional notes. |
| **Occurrence** | One instance of a task becoming due (e.g. "Clean rabbit cages, Wed 11 Nov"). A one-off task has exactly one occurrence. |
| **Day / week** | Based on the user's local time zone. A week runs **Monday to Sunday**. |
| **Priority** | Chosen by the user: **High, Normal, Low, None**. |
| **Overdue** | A *status* the app calculates, not a priority. An occurrence is overdue when its due period has passed and it hasn't been completed. |
| **Missed** | An overdue occurrence that was replaced by the task's next occurrence before being completed. It is recorded in history and removed from the list. |

### Schedule types

| # | Schedule type | Example | When due | Can it be overdue? |
|---|---|---|---|---|
| 1 | **One-off** | Upload CSE1002 assignment, Fri 13 Nov 2026 | On the due date | Yes. It stays overdue until completed or deleted, because there is no next occurrence. |
| 2 | **Daily** | Take creatine | Every day | No. An uncompleted day is recorded as missed at midnight. |
| 3 | **Specific weekdays** | Clean rabbit cages (Wed, Sat); Clean room (Sun) | On each selected weekday | Yes, from the day after until the next scheduled day, when it becomes missed. |
| 4 | **Weekly target** | Go to the gym, 5 times per week | Any days, Mon to Sun, until the target count is reached | No. If the target isn't reached by the end of Sunday, the week is recorded as missed and the count resets on Monday. |
| 5 | **Monthly window** | Clean rabbit tray, first 7 days of each month | Any day within the first *N* days of each calendar month | Yes, from the end of the window until the next month's window starts, when it becomes missed. |

## 5. User stories and acceptance criteria

Priorities use **MoSCoW**: **Must** (v1 can't ship without it), **Should** (planned for v1, but can slip to a fast follow-up), **Could** (only if time allows), **Won't** (see section 7).

### Epic A: Accounts

**US-01: Sign up** (Must)
As a new user, I want to create an account so that my tasks are saved and private to me.
- Given I'm on the sign-up page, when I enter a valid email and a password that meets the minimum rules, then my account is created and I'm logged in.
- Given an email that's already registered, when I try to sign up, then I see an error and no duplicate account is created.
- Given an invalid email or weak password, when I submit, then I see a clear message explaining what to fix.

**US-02: Log in and out** (Must)
As a user, I want to log in and out so that I can use the app on my phone and protect my data.
- Given correct credentials, when I log in, then I'm taken to the Today screen.
- Given incorrect credentials, when I log in, then I see a generic "email or password is incorrect" message (it doesn't reveal which one was wrong).
- Given I'm logged in, when I log out, then I return to the login page and can't see my tasks until I log in again.
- Given I've logged in on my phone, when I reopen the app later, then I'm still logged in (until I log out or my session expires).

**US-03: Private data** (Must)
As a user, I want my tasks to be visible only to me so that my personal information stays private.
- Given two users A and B, when user A views, edits or deletes tasks, then they can only ever access their own tasks, including through direct requests to the server.

### Epic B: Managing tasks

**US-04: Create a one-off task** (Must)
As a user, I want to create a task with a single due date so that I can track deadlines like assignments.
- Given the create-task form, when I enter a name and due date and choose "One-off", then the task is saved.
- I can set a priority (High / Normal / Low / None, default None).
- I can set when it **starts showing**: *always*, *on the due date*, or *N days before the due date*.
- Given a task with no name, when I try to save it, then I see a validation error.

**US-05: Create a daily task** (Must)
As a user, I want to create a task that repeats every day so that I never rewrite habits like taking creatine.
- Given I choose "Daily", when I save, then the task appears on the Today screen every day without me doing anything else.

**US-06: Create a specific-weekdays task** (Must)
As a user, I want a task to repeat on chosen weekdays so that chores like cleaning the rabbit cages on Wednesday and Saturday appear on the right days.
- Given I choose "Specific weekdays", I must select at least one day (Mon to Sun).
- Given a task set to Wed and Sat, then it appears on the Today screen on Wednesdays and Saturdays only (unless it's overdue).

**US-07: Create a weekly-target task** (Must)
As a user, I want to set a task I must do a number of times per week so that I can track goals like going to the gym 5 times.
- Given I choose "Weekly target", I must enter a target from 1 to 7.
- The task shows its progress on the Today screen (e.g. "Go to the gym, 3/5 this week").
- Ticking it off adds 1 to this week's count, at most once per day.
- Once the target is reached, it no longer appears in the Today list for the rest of that week.

**US-08: Create a monthly-window task** (Must)
As a user, I want a task that needs doing once within the first few days of each month so that chores like cleaning the rabbit tray get done without a fixed day.
- Given I choose "Monthly window", I enter the window length *N* in days (default 7).
- The task appears on the Today screen from day 1 to day *N* of each calendar month until it's completed.

**US-09: Add optional notes** (Must)
As a user, I want to add notes to a task so that I can store details such as "use the blue tray liner" or a submission link.
- Notes are optional and can be added or changed when creating or editing a task.
- Notes are visible when I open a task's details.

**US-10: Edit a task** (Must)
As a user, I want to edit a task so that I can fix mistakes or change its schedule.
- I can change any field: name, schedule, priority, show-from setting, notes.
- Changing a schedule affects future occurrences only. Past history stays as it was.

**US-11: Delete a task** (Must)
As a user, I want to delete a task I no longer need.
- Given I choose delete, I'm asked to confirm before it is removed.
- After deleting, the task no longer appears on the Today screen or in future views.

**US-12: Tick off and untick** (Must)
As a user, I want to tick off a task when I've done it, and undo it if I tapped by mistake.
- Given an occurrence on the Today screen, when I tick it, then it's marked complete with today's date and moves out of the to-do list.
- Given a task I completed today, when I untick it, then it returns to the list in its normal position (and a weekly-target count goes down by 1).
- Ticking off an overdue occurrence marks that occurrence complete (recorded as done late).

### Epic C: Today screen

**US-13: See what matters now** (Must)
As a user, I want to open the app and immediately see the tasks that matter right now.
- The Today screen shows:
  - all **overdue** occurrences;
  - tasks **due today** (daily, specific weekdays scheduled for today, one-offs due today);
  - **one-off** tasks whose "show from" setting says they should be visible today;
  - **monthly-window** tasks while the window is open and they're incomplete;
  - **weekly-target** tasks until this week's target is reached.
- Tasks that don't match any of these are not shown.
- Tasks completed today are shown in a separate, collapsed "Done today" section so they can be unticked.

**US-14: Sorted by importance** (Must)
As a user, I want the list sorted so that the most important things are at the top.
- Sections appear in this order: **Overdue → High → Normal → Low → None.**
- Inside the Overdue section, tasks are sorted by their priority (High to None).
- Inside each section, tasks are sorted by schedule type: **One-off → Daily → Specific weekdays → Weekly target → Monthly window.**
- Remaining ties are broken by due date (earliest first), then alphabetically by name.

### Epic D: History

**US-15: Record history** (Must)
As a user, I want every occurrence's result recorded so that I have an accurate record of what I did and didn't do.
- Each occurrence is recorded as **done** (with the completion date) or **missed**, following the rules in section 4.
- History is recorded from the day a task is created, even before the history view exists.

**US-16: View a past day** (Should)
As a user, I want to pick a past date on a calendar and see what was and wasn't done that day so that I can look back on my week.
- Given I select a past date, then I see every occurrence that was due that day.
- Incomplete (missed or overdue) occurrences appear first, then completed ones.
- Future dates can't be selected in this view.

### Epic E: Weekly view

**US-17: Weekly calendar view** (Should)
As a user, I want to see the current week (Mon to Sun) laid out by day so that I can plan ahead.
- The view shows each day of the current week with the occurrences scheduled for that day.
- Weekly-target and monthly-window tasks show their progress or window instead of a fixed day.
- I can move to the previous or next week.

## 6. Non-functional requirements

| Category | Requirement |
|---|---|
| **Platform** | A web app built with HTML, CSS and JavaScript, used in a mobile browser. No native app. |
| **Responsive design** | Mobile-first. Fully usable on a 360px-wide screen. Tap targets at least 44×44px. Also works on a laptop browser. |
| **Browser support** | Current versions of Safari (iOS), Chrome (Android) and Chrome/Firefox/Edge on desktop. |
| **Performance** | The Today screen loads and is usable within 2 seconds on a typical 4G connection. Ticking a task updates the screen instantly (under 200 ms). |
| **Security** | All traffic over HTTPS. Passwords are never stored in plain text (salted hashing, or delegated to a trusted auth provider). The server checks every request so users can only access their own data. User input is validated and escaped to prevent XSS and injection. |
| **Privacy** | Collect only what's needed: email, password (hashed) and task data. No analytics or third-party tracking in v1. |
| **Reliability** | Data persists across devices and sessions. No completed or missed record is lost when the app is closed. |
| **Dates and time zones** | "Today", week and month boundaries are based on the user's local time zone (default Australia/Melbourne). Behaviour around midnight and daylight saving changes is covered by tests. |
| **Accessibility** | Aim for the basics of WCAG 2.1 AA: semantic HTML, labelled form fields, usable by keyboard, and colour is never the only way to show status (e.g. overdue also has a text label). |
| **Cost** | $0 to run: free-tier hosting and database only. |
| **Quality / process** | Code lives on GitHub with feature branches and pull requests. Automated tests and linting run in GitHub Actions on every PR. Core logic (recurrence, overdue/missed, sorting) has unit tests. |
| **Maintainability** | Scheduling and sorting logic is kept separate from the UI so it can be tested on its own. |

## 7. Out of scope and future ideas

These were discussed and deliberately left out of v1 to keep it achievable for one part-time developer.

| Idea | Why it's not in v1 |
|---|---|
| Reminders and push notifications | Significant extra work for web apps (service workers, permissions, iOS limits). The core need is met by opening the app and checking. |
| Shared lists / households | Adds permissions and sync complexity. v1 is one user, private tasks. |
| Pausing recurring tasks (e.g. holidays) | Nice to have. Can be worked around by editing or deleting a task. |
| Streaks, stats and charts | Not a goal for this product. History data is recorded, so these could be added later. |
| Categories / tags | Not needed with the current number of tasks. |
| Subtasks / checklists | Adds complexity to completion logic. |
| Dark mode | Cosmetic; can be added once the UI is stable. |
| Offline use | Requires caching and conflict handling. |
| Other recurrence patterns (every 4 weeks, every N days, custom rules) | The five schedule types cover all current tasks. |
| Password reset by email | Needs an email-sending service. Revisit if the chosen auth approach doesn't include it for free. |
| Native mobile app | The web app in a mobile browser is enough. |

## 8. Assumptions, constraints and open questions

### Assumptions
- Users are individuals managing tens of tasks, not hundreds.
- Users have an internet connection when using the app.
- A weekly-target task can be ticked off at most once per day.

### Constraints
- One developer, part-time, with an unpredictable weekly schedule. Scope is kept to the smallest version that solves the core problem.
- Must be free to host and run.
- Built with HTML and JavaScript as the main learning focus. CSS styling can be AI-assisted.
- Technology choices (framework, backend, database, hosting, auth) will be made in the technical design doc and recorded as ADRs.

### Open questions
1. Should deleting a task also delete its history, or keep the history visible in the past-day view?
2. When editing a recurring task's schedule, should any current overdue occurrence be kept or cleared?

## 9. Release plan

Because my weekly availability is unpredictable, v1 is split into small releases that each leave me with something usable.

| Release | Contents | Done when |
|---|---|---|
| **MVP** | Epics A to C, US-15 (history recorded, no view yet). All **Must** stories. | I can set up all my Appendix A tasks and use the Today screen daily. |
| **v1.0** | US-16 (past-day view) and US-17 (weekly view). All **Should** stories. | The success metric is met: one week of use instead of paper. |
| **Later** | Items from section 7, prioritised after the v1 retro. | |

Work is planned in 1 to 2 week sprints from a GitHub Issues backlog, with one issue per user story (split into smaller tasks where needed).

---

## Appendix A: Example tasks

| Task | Schedule type | Details |
|---|---|---|
| Take creatine | Daily | |
| Do skincare | Daily | |
| Clean rabbit cages | Specific weekdays | Wednesday, Saturday |
| Clean room | Specific weekdays | Sunday |
| Upload CSE1002 assignment | One-off | Due Fri 13 Nov 2026 |
| Go to the gym | Weekly target | 5 times per week |
| Clean rabbit tray | Monthly window | First 7 days of each month |
