# The Observatory: site review findings (2026-11-24)

Read-only review by Happy Path. Each item: what we saw, where, and what 'fixed' looks like. `check` passes while the bug is present.

## OBS-01 · Issue pages have no links to the sources  
**Priority:** fix first  
**Pages:** https://observatory.mgorsky.net/archive/471/, https://observatory.mgorsky.net/archive/472/

**What we saw:** Recent issue pages name a source as plain text ("TechCrunch; also Bloomberg Technology", "arXiv · AI research") and the story titles are not links. External links inside the issue body: /archive/472 0 (20 stories), 473 0 (12), 477 0 (12), 450 0 (20), 462 0 (20), 451 0 (20), 446 0 (12 papers), 169 0 (6). The oldest issue, /archive/6 (29 Sep), still has 80 links for 40 stories, and the Notes essay /archive/471 has 48. Terms: "Every note links to its source, and you should read the source before you rely on a claim." Privacy: "each with a link to the original." For Science corner papers there is no link to the paper anywhere: on the front page the 5 paper titles link to /archive/446/, and that page links nowhere.

**Steps:**
1. Open /archive/472/ (the "AI daily, 24-Nov-26 →" link on the front page)
2. Try to open the source of any story, for example "Sierra raises $1.75B Series F" (TechCrunch). Neither the title nor the source name is a link.
3. Compare /archive/6/, where every story title is a link

**Suggested fix:** Render the story title (or the source name) as a link to the primary source URL in the web view of an issue, as in the September template and on the front page. For Science corner, link the arXiv page.

**Fixed when:** each story on an issue page links to its original, as /archive/6/ does. Each Science corner paper links to the paper.

**Check:**
```json
{
 "url": "https://observatory.mgorsky.net/archive/472/",
 "viewport": "desktop",
 "assert_bug_present": [
  {
   "text_present": "TechCrunch; also Bloomberg Technology"
  },
  {
   "js_true": "document.querySelectorAll('article.issue-body span.anchor').length>=10 && [...document.querySelectorAll('article.issue-body a[href]')].filter(a=>!a.href.includes('mgorsky.net')).length===0"
  }
 ]
}
```

## OBS-02 · A source line and an "author has a stake" note are missing  
**Priority:** fix first  
**Pages:** https://observatory.mgorsky.net/ai-in-management/, https://observatory.mgorsky.net/archive/451/

**What we saw:** On / and /ai-in-management/ (EN), "Products / Joanna Stern says Siri AI still fails basics on the HomeView despite Apple's Google partnership" has no source line; the next story follows at once. /ru/ shows "Bloomberg Tech · у автора есть доля: …" and /archive/451 shows "Bloomberg Tech · at 17:08 · author has a stake: Joanna Stern is arguing from her own HomeView review …". The "author has a stake" warning matters most and is the part that is missing.

**Steps:**
1. Open /, scroll to "PODCASTS AND TALKS ABOUT AI, 22-NOV-26"
2. Compare with the same item on /ru/

**Suggested fix:** Check the EN teaser template for items with a stake label (the source line seems to be dropped when the label is present or too long).

**Fixed when:** the English front page shows "Bloomberg Tech · author has a stake: …" under this story, as the Russian page does.

**Check:**
```json
{
 "url": "https://observatory.mgorsky.net/",
 "viewport": "desktop",
 "assert_bug_present": [
  {
   "js_true": "document.body.innerText.includes(\"Google partnership\\nCompanies / Anthropic raises\")"
  }
 ]
}
```

## OBS-03 · A placeholder instead of a number on the About page  
**Priority:** fix first  
**Pages:** https://observatory.mgorsky.net/about/, https://observatory.mgorsky.net/ru/about/

**What we saw:** Step "05 · CHECKING / The credibility check": the big number is the literal text "{labeled}", followed by "of 1,170 notes this week passed with a label stating their origin". Same on /ru/about/ ("{labeled} заметок за эту неделю прошли проверку"). Every other step shows a number (5,730 / 11 / 2,648 / 178 / 8,282 / 11). The same page also shows "Explainer video: coming".

**Steps:**
1. Open /about/ and scroll to "05 · CHECKING"
2. Open /ru/about/ and scroll to "05 · ПРОВЕРКА"

**Suggested fix:** Fill the {labeled} variable in the About template (the value is probably missing from the stats object), and hide a step whose value is missing instead of printing the token.

**Fixed when:** the card shows a number, or the card hides its number when the value is missing. Same on /ru/about/.

**Check:**
```json
{
 "url": "https://observatory.mgorsky.net/about/",
 "viewport": "desktop",
 "assert_bug_present": [
  {
   "text_present": "{labeled}"
  },
  {
   "text_present": "of 1,170 notes this week passed with a label stating their origin"
  }
 ]
}
```

## OBS-04 · Story counts are higher than the stories shown  
**Priority:** should fix  
**Pages:** https://observatory.mgorsky.net/archive/

**What we saw:** Archive / group-page count vs stories on the issue page: AI daily 24-Nov (/archive/472) "28 stories" vs 20; AI daily 22-Nov (450) "37 stories" vs 20; India & AI 24-Nov (477) "18 stories" vs 12; Ethiopia daily brief 24-Nov (473) "14 stories" vs 12; AI daily 23-Nov (462) "23 stories" vs 20. Matching: Podcasts 22-Nov (451) 20/20, Ethiopia AI weekly 21-Oct (169) 6/6, 29 Sep (6) 40/40. Several issues show exactly 20 stories, so the page seems to be capped at 20 while the count includes all candidates.

**Steps:**
1. Open /archive/ and read "AI daily, 22-Nov-26 … 37 stories"
2. Open it (/archive/450/) and count the stories under the topic headings (20)

**Suggested fix:** Compute "N stories" from the stories actually rendered in the issue (after the cap or selection), or show the rest under a "more stories" list on the issue page.

**Fixed when:** each count equals the stories on its issue page, or the issue page lists the rest.

**Check:**
```json
{
 "url": "https://observatory.mgorsky.net/archive/450/",
 "viewport": "desktop",
 "assert_bug_present": [
  {
   "js_true": "fetch('/archive/').then(r=>r.text()).then(h=>/Raising About \\$119\\.5 Billion Gross[\\s\\S]{0,400}?37 stories/.test(h) && document.querySelectorAll('article.issue-body span.anchor').length===20)"
  }
 ]
}
```

## OBS-05 · The Anthropic IPO figures need context  
**Priority:** minor  
**Pages:** -

**What we saw:** Front page, AI in Management: lead "pricing its shares at $83 … The offering of 980 million shares was the largest IPO on record" (980M × $83 = $81.3B), then "Anthropic raises $107.3 billion in IPO at $83 per share" (implies 1.29B shares), then "Anthropic CFO says company has raised almost $120 billion in IPO". Group page issue leads: 12-Nov "seeks up to $90.4 billion", 16-Nov "Anthropic's $1.4 trillion IPO" (a valuation called an IPO), 22-Nov "Raising About $119.5 Billion Gross" (IPO and private placement). Podcast issue 451: "implied valuation is near $2 trillion"; 8-Nov "targeting $1.5tn valuation". None of these says how the figures relate.

**Steps:**
1. Open / and read the three Anthropic items in AI in Management
2. Open /ai-in-management/ and read the Every issue leads for 12-Nov, 16-Nov, 19-Nov, 22-Nov

**Suggested fix:** In the credibility or editing step, flag numeric conflicts between notes on the same event and add the qualifier ("including the private placement", "valuation"). Fix the 16-Nov headline wording.

**Fixed when:** each figure says what it includes ("including the private placement"), and valuation is not called IPO size.

**Check:**
```json
{
 "url": "https://observatory.mgorsky.net/",
 "viewport": "desktop",
 "assert_bug_present": [
  {
   "text_present": "The offering of 980 million shares"
  },
  {
   "text_present": "Anthropic raises $107.3 billion in IPO at $83 per share"
  },
  {
   "text_present": "raised almost $120 billion in IPO"
  }
 ]
}
```

## OBS-06 · "Notes on Agentic Work" has two schedules  
**Priority:** should fix  
**Pages:** https://observatory.mgorsky.net/agentic-software-engineering/, https://observatory.mgorsky.net/ru/, https://observatory.mgorsky.net/subscribe/

**What we saw:** Group page: "Notes on Agentic Work · Mon, Wed, 18:00" and next line "AI agents writing software, as a dense letter every Tuesday and Thursday". The same pair is on /subscribe/ ("Mon, Wed" beside "every Tuesday and Thursday"), the front page says "· Mon, Wed", /ru says "пн, ср". The latest issue (/archive/471) is headed "NOTES ON AGENTIC WORK — TUESDAY, 24 NOVEMBER 2026". Older issues in the group are dated Fri 2, 9, 16, 23, 30 Oct, Wed 28 Oct and Tue 10 Nov. The 4 newest have no date in their archive title ("Software Work Needs Evidence Beyond Passing Tests"), unlike every other publication.

**Steps:**
1. Open /agentic-software-engineering/, read "The publications"
2. Open /subscribe/, read the Notes on Agentic Work row
3. Open /archive/471/ and read the date line

**Suggested fix:** Fix either the schedule label or the description. Put the date in the archive title of Notes issues ("Notes on Agentic Work, 24-Nov-26 · Software Work Needs …").

**Fixed when:** one schedule appears everywhere and matches the days issues go out.

**Check:**
```json
{
 "url": "https://observatory.mgorsky.net/agentic-software-engineering/",
 "viewport": "desktop",
 "assert_bug_present": [
  {
   "text_present": "Mon, Wed, 18:00"
  },
  {
   "text_present": "every Tuesday and Thursday"
  }
 ]
}
```

## OBS-07 · "Dispatches from Others" is offered, but its last issue was 2 November  
**Priority:** should fix  
**Pages:** https://observatory.mgorsky.net/about/, https://observatory.mgorsky.net/archive/, https://observatory.mgorsky.net/subscribe/

**What we saw:** /subscribe/ offers "Interesting Politics · Dispatches from Others … Sun–Fri" with hours 09:00–21:00. /about/: "The Observatory has four desks: AI, Ethiopia, India and Interesting Politics". /archive/ filter "Dispatches from Others": newest issue "Dispatches from Others, 2-Nov-26", 22 days ago, while every other daily publication has an issue on 24-Nov. The front page menu and the topic tabs list only 5 groups, with no Interesting Politics page. The lead "OUR COUNTRY AGAINST THE WORLD" repeats on 10 issues in a row (15–27 Oct). Terms and Privacy describe the service as "about artificial intelligence and Ethiopia" (no India, no politics desk).

**Steps:**
1. Open /subscribe/, find "Dispatches from Others · Sun–Fri"
2. Open /archive/, click the "Dispatches from Others" filter, read the newest date (2-Nov-26)

**Suggested fix:** Mark it paused ("paused since 2 Nov") or remove it from /subscribe/ and from the About desk count. If it is active, find out why it stopped sending.

**Fixed when:** it shows new issues again, or it is marked paused, or it leaves /subscribe/.

**Check:**
```json
{
 "url": "https://observatory.mgorsky.net/archive/",
 "viewport": "desktop",
 "assert_bug_present": [
  {
   "js_true": "(()=>{const m=document.body.innerText.match(/Dispatches from Others, \\d+-\\w+-\\d+/g); return !!m && m[0]==='Dispatches from Others, 2-Nov-26'})()"
  },
  {
   "js_true": "fetch('/subscribe/').then(r=>r.text()).then(h=>h.includes('Dispatches from Others'))"
  }
 ]
}
```

## OBS-08 · Some days are off the stated schedule  
**Priority:** minor  
**Pages:** https://observatory.mgorsky.net/archive/

**What we saw:** Science corner ("Friday"): issues 28-Oct (Wednesday) and 30-Oct; none on 6-Nov (Friday). Ethiopian banking and fintech ("Friday"): 28-Oct and 30-Oct. Agentic Software Engineering: 28-Oct and 30-Oct. AI daily and Ethiopia daily brief ("Sun–Fri"): 5-Nov then 8-Nov, no Friday 6-Nov. The site gives no note about the missing day.

**Steps:**
1. Open /ai-science-corner/ and read the issue dates
2. Open /ai-in-management/ and look for AI daily 6-Nov-26

**Suggested fix:** Show a short notice on the archive for missed days, and check why the weekly publications ran on 28 Oct.

**Fixed when:** issues follow the stated days, or the archive notes a missed day.

**Check:**
```json
{
 "url": "https://observatory.mgorsky.net/ai-science-corner/",
 "viewport": "desktop",
 "assert_bug_present": [
  {
   "text_present": "Science corner · Friday"
  },
  {
   "text_present": "Science corner, 28-Oct-26"
  },
  {
   "js_true": "!document.body.innerText.includes('Science corner, 6-Nov-26')"
  }
 ]
}
```

## OBS-09 · An issue page doesn't say which issue it is  
**Priority:** should fix  
**Pages:** https://observatory.mgorsky.net/archive/264/

**What we saw:** On 11 of 12 issue pages checked (all but the Notes essay 471), the body starts with "← The archive", then the group link and bullets. The issue name and date ("Ethiopia daily brief, 29-Oct-26") appear only in the browser tab title. The only visible date is the masthead "Tuesday, 24 November 2026", so an old issue reads as today's. There is no previous/next link between issues of a publication, only "← The archive".

**Steps:**
1. Open /archive/264/ (Ethiopia daily brief, 29-Oct-26)
2. Look for the issue date on the page, then look for a way to the next or previous brief

**Suggested fix:** Render the issue title as the page h1 (this also fixes the known missing-h1 item) and add prev/next links by publication.

**Fixed when:** each issue starts with its name and date, with previous and next links. That heading would also give these pages the main heading they lack (see the last section).

**Check:**
```json
{
 "url": "https://observatory.mgorsky.net/archive/264/",
 "viewport": "desktop",
 "assert_bug_present": [
  {
   "js_true": "document.title.startsWith('Ethiopia daily brief, 29-Oct-26') && !document.body.innerText.includes('29-Oct-26') && !/29 October/.test(document.body.innerText) && ![...document.querySelectorAll('a')].some(a=>/previous|next|older|newer/i.test(a.innerText))"
  }
 ]
}
```

## OBS-10 · Old leads come back, and Science corner entries look the same  
**Priority:** minor  
**Pages:** https://observatory.mgorsky.net/ai-science-corner/, https://observatory.mgorsky.net/archive/, https://observatory.mgorsky.net/ethiopia/

**What we saw:** "Awash Bank applies for Djibouti banking licence, weighs Awash Group entity" leads Ethiopia daily brief 2-Nov and Ethiopian banking and fintech 13-Nov (11 days later). "National Bank of Ethiopia issues Directive No. FS/01/2026 launching a regulatory sandbox" leads the daily brief 18-Nov and banking 20-Nov. "Anthropic Files for IPO on Nasdaq Under Ticker ANT" leads both AI daily and Podcasts on 28-Oct, and "Google DeepMind's Keerthana Gopalakrishnan on Gemini Robotics 2" leads both on 4-Oct. On /ai-science-corner/ and /archive/, all 8 Science corner issues read "Research papers of the week on AI and the fields around it, in plain English: …" with no lead or count, so a reader cannot tell them apart.

**Steps:**
1. Open /ethiopia/, find "Awash Bank applies for Djibouti" twice in "Every issue"
2. Open /ai-science-corner/ and read the 8 issue entries

**Suggested fix:** Exclude notes already used as the lead of another issue in the last N days from lead selection, or label them as follow-ups. Use the top paper title as the Science corner archive lead.

**Fixed when:** a repeat lead is marked ("first reported 2 Nov") or skipped, and each Science corner entry shows its top paper.

**Check:**
```json
{
 "url": "https://observatory.mgorsky.net/ethiopia/",
 "viewport": "desktop",
 "assert_bug_present": [
  {
   "js_true": "document.body.innerText.split('Awash Bank applies for Djibouti banking licence').length-1===2 && document.body.innerText.split('Directive No. FS/01/2026 launching a regulatory sandbox').length-1>=2"
  }
 ]
}
```

## OBS-11 · The Ethiopia AI weekly is short and repeats itself  
**Priority:** should fix  
**Pages:** https://observatory.mgorsky.net/archive/169/

**What we saw:** Issue counts: 18-Nov "3 stories", 11-Nov "1 stories", 4-Nov "2 stories". /archive/169 (21-Oct, 6 stories): "Ethiopia Seeks Partner Support for AI Institutions at Bangkok Roundtable" (Fana, "Tigist Fisseha") and "Ethiopia Joins AI Roundtable at IMF and World Bank Meetings" (Ministry of Finance, "Tiguist Fisseha") are the same event with the name spelled two ways. "Ethiopia Ties AI and Digital Skills to Jobs Agenda" (Fana) and "Ethiopia Pursues Private Financing for AI…" (Birr Metrics) are the same speech. "ThatsApp — OpenGradient Announces Decentralized AI Model Hub With Bittensor and Nvidia" (Telebirr Telegram) is not about Ethiopia, yet it is the lead shown for this issue in /archive/ and on /ethiopia/. About promises: "Accounts of the same event are compared and the best is chosen". Also "1 stories" grammar (2 places), and a "Ethiopia daily brief, 29-Oct-26" with 1 story (a company statement).

**Steps:**
1. Open /ethiopia/ and read the "Every issue" counts for Ethiopia AI weekly
2. Open /archive/169/ and compare the Fana and Ministry of Finance stories, and the Fana and Birr Metrics stories

**Suggested fix:** Run the same-event merge across sources inside weekly issues; apply the desk scope rule to Telegram items (OpenGradient); set a minimum story count for a weekly issue; use plural rules ("1 story").

**Fixed when:** one event appears once per issue, leads match the desk's topic, and counts read "1 story".

**Check:**
```json
{
 "url": "https://observatory.mgorsky.net/archive/169/",
 "viewport": "desktop",
 "assert_bug_present": [
  {
   "text_present": "Tigist Fisseha"
  },
  {
   "text_present": "Tiguist Fisseha"
  },
  {
   "text_present": "ThatsApp — OpenGradient"
  },
  {
   "js_true": "fetch('/ethiopia/').then(r=>r.text()).then(h=>h.includes('1 stories'))"
  }
 ]
}
```

## OBS-12 · The Russian lists differ from the English ones  
**Priority:** should fix  
**Pages:** https://observatory.mgorsky.net/agentic-software-engineering/, https://observatory.mgorsky.net/ru/agentic-software-engineering/

**What we saw:** Same issue ids, different leads. EN /agentic-software-engineering/: 471 "OpenAI says Codex found the root cause …", 426 "Harvard working paper finds …", 344 "Agentic Software Engineering, 10-Nov-26 · OpenAI releases GPT-6 Cyber …". RU: 471 "CodeRabbit launches public beta of Verify …", 426 "OpenAI releases GPT-6.2 Astra and GPT-6.2 Sol …", 344 "…, 9 нояб. 2026 г. · OpenAI builds Pando Git backend …". Only 74 (giflib) matches. 5 of 11 RU leads are untranslated English. The RU front page shows 5 English stories in this section ("CodeRabbit launches …", "AWS Releases …", "Release · InfoQ …"), while the EN front page shows no stories in this section, only the issue link.

**Steps:**
1. Open /agentic-software-engineering/ and /ru/agentic-software-engineering/ side by side, compare "Every issue" / "ВСЕ ВЫПУСКИ"
2. Open /ru/ and look at the Agentic Software Engineering section

**Suggested fix:** Build the RU group list from the same lead story as EN (translated), and take the issue date from one field. Translate the front-page teaser stories, or hide them on /ru as on EN.

**Fixed when:** each issue has the same lead and date in both languages, translated.

**Check:**
```json
{
 "url": "https://observatory.mgorsky.net/ru/agentic-software-engineering/",
 "viewport": "desktop",
 "assert_bug_present": [
  {
   "text_present": "CodeRabbit launches public beta of Verify"
  },
  {
   "js_true": "fetch('/agentic-software-engineering/').then(r=>r.text()).then(h=>h.includes('Agentic Software Engineering, 10-Nov-26') && h.includes('OpenAI says Codex found the root cause') && document.body.innerText.includes('9 нояб. 2026'))"
  }
 ]
}
```

## OBS-13 · English words left on Russian pages  
**Priority:** minor  
**Pages:** https://observatory.mgorsky.net/ru/, https://observatory.mgorsky.net/ru/archive/, https://observatory.mgorsky.net/ru/india/

**What we saw:** /ru/archive/: "18 stories", "8 stories" … (132 English counts, no Russian ones). /ru/india/, /ru/agentic-software-engineering/: H2 "The publications". /ru/: "Industry / Отчет Tracxn …" (2 English category labels; other categories are translated, "Компании /"), "Модели visión-язык плохо угадывают слова" (Spanish word, also on /ru/ai-science-corner/), "TechCrunch; also Bloomberg Technology", "Видео · AI daily, 24-Nov-26". The About step table on /ru/about/ is fully English ("Relevance · X posts … relevant posts lost").

**Steps:**
1. Open /ru/archive/ and read any issue count
2. Open /ru/ and find "Industry /" in the India section and "visión" in the Science section

**Suggested fix:** Add the missing keys to the RU string table (stories count with Russian plural forms, "The publications", category "Industry", "also"); add a check that flags Latin-script words in RU headlines other than names.

**Fixed when:** these strings are Russian, with Russian plural forms for the counts.

**Check:**
```json
{
 "url": "https://observatory.mgorsky.net/ru/",
 "viewport": "desktop",
 "assert_bug_present": [
  {
   "text_present": "visión"
  },
  {
   "text_present": "Industry / Отчет Tracxn"
  },
  {
   "js_true": "fetch('/ru/archive/').then(r=>r.text()).then(h=>(h.match(/\\d+ stories/g)||[]).length>50)"
  }
 ]
}
```

## OBS-14 · The Russian logo is cut off on a phone  
**Priority:** minor  
**Pages:** https://observatory.mgorsky.net/ru/

**What we saw:** At 390px, the masthead logo link on /ru/ measures 403px (left -6.6, right 396.6), so the "t" of "the" and the "y" of "observatory" are clipped. On the EN front page the same logo fits (6.4 to 383.6). The RU masthead also puts the menu above the logo, unlike EN.

**Steps:**
1. Open /ru/ on a 390px-wide phone
2. Look at the masthead

**Suggested fix:** Use the same logo size rule as the EN masthead (the RU layout seems to use a different max-width/font-size), for example width:min(100%, …).

**Fixed when:** the logo fits inside the screen on /ru/, as on /.

**Check:**
```json
{
 "url": "https://observatory.mgorsky.net/ru/",
 "viewport": "mobile",
 "assert_bug_present": [
  {
   "js_true": "(()=>{const r=document.querySelector('a[aria-label=\"The Observatory\"]').getBoundingClientRect(); return r.left<0 || r.right>innerWidth})()"
  }
 ]
}
```

## OBS-15 · Small filter buttons and broken dates on the phone archive  
**Priority:** minor  
**Pages:** https://observatory.mgorsky.net/archive/

**What we saw:** The 10 filter buttons ("All", "India & AI" … "Dispatches from Others") are 34px tall at 390px. The filters work and the selected chip is clearly filled. Issue titles wrap inside the date: "Dispatches from Others, 2-" on one line and "Nov-26" on the next; "Dispatches from Others, 1-Nov-" / "26".

**Steps:**
1. Open /archive/ on a 390px phone
2. Tap "Dispatches from Others" and look at the first entries

**Suggested fix:** min-height:44px on .filters button; wrap the date part of the title in a nowrap span.

**Fixed when:** the buttons are about 44 px tall and each date stays on one line.

**Check:**
```json
{
 "url": "https://observatory.mgorsky.net/archive/",
 "viewport": "mobile",
 "assert_bug_present": [
  {
   "js_true": "(()=>{const b=[...document.querySelectorAll('.filters button')]; return b.length>=9 && b.every(x=>x.getBoundingClientRect().height<44)})()"
  }
 ]
}
```

## OBS-16 · The visitor counter stops with an error  
**Priority:** should fix  
**Pages:** https://observatory.mgorsky.net/about/

**What we saw:** the page's script stops with goatcounter.count is not a function. Nothing on the screen shows it, so there is no screenshot. The Privacy page says the site counts visits with GoatCounter.

**Fixed when:** the error is gone from the browser console and visits appear in GoatCounter.

## OBS-17 · The Privacy page slides sideways on a phone  
**Priority:** should fix  
**Pages:** https://observatory.mgorsky.net/privacy/

**What we saw:** a long web address is wider than the screen, so the page moves left and right.

**Fixed when:** long addresses wrap and the page no longer scrolls sideways.

## OBS-18 · Four pages have no main heading  
**Priority:** minor  
**Pages:** https://observatory.mgorsky.net/archive/472/, https://observatory.mgorsky.net/ru/, https://observatory.mgorsky.net/ru/archive/472/

**What we saw:** these pages have no top-level heading. On the front page the logo picture is the only title. Screen readers and search engines use that heading to tell what a page is. It is not visible, so there is no screenshot.

**Fixed when:** each page has one main heading (card 9's issue heading would cover the issue pages).
