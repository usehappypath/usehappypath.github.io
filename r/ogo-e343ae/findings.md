# OGO: site review findings (24–25 November 2026)

Read-only review of https://www.ogo.co.il by Happy Path. Each item: what we saw, where, and what 'fixed' looks like. `check` passes while the bug is present.

## OBS-01 · Search results do not say what you searched for, and an emoji search shows everything  
**Priority:** should fix  
**Viewport:** desktop and mobile  
**Pages:** https://www.ogo.co.il/ru/filtered?q=%D1%85%D0%B0%D0%BD%D1%83%D0%BA%D0%B0, https://www.ogo.co.il/ru/filtered?q=%F0%9F%8E%B5%F0%9F%8E%89

**What we saw:** After a search for "ханука" (Hanukkah) the list heading reads "Все события в Израиле" (all events in Israel), as on the home page, and the page heading is the site name, "OGO! — афиша событий в Израиле". The query appears only inside the search box, and there is no result count. A search for an emoji ("🎵🎉") shows the full event list with no "nothing found" message. A text search with no match does say "По запросу «…» ничего не нашлось".

**Why it matters:** A visitor cannot tell a list of results from the normal listing, so an empty search looks like a long list of matches.

**Steps:**
1. Type "ханука" in the "Найдите своё событие" box on /ru and press Enter.
2. Read the heading above the results.
3. Repeat with "🎵🎉".

**Suggested fix:** Add the query as a removable chip in the heading line, with a result count. If the normalised query is empty but the raw q is not, render the no-results state with the raw query quoted.

**Fixed when:** The results page names the query and the number of results ("Результаты по «ханука»: N событий") with a way to clear it, and a query that matches nothing shows the "ничего не нашлось" message.

**Check:**
```json
{
 "url": "https://www.ogo.co.il/ru/filtered?q=%F0%9F%8E%B5%F0%9F%8E%89",
 "viewport": "desktop",
 "wait_ms": 15000,
 "assert_bug_present": [
  {
   "js_true": "!document.body.innerText.includes('ничего не нашлось') && document.querySelectorAll('a[href*=\"/event/\"]').length>10"
  }
 ]
}
```

## OBS-02 · The Lumagica light festival is filed under Theatre and search cannot find it  
**Priority:** should fix  
**Viewport:** desktop  
**Pages:** https://www.ogo.co.il/ru/event/il/tel-aviv/2026/11/25/light-festival-lumagica-sg3haMfwLL4yDeJsz22a12, https://www.ogo.co.il/ru/festivals, https://www.ogo.co.il/ru

**What we saw:** The breadcrumb says "Тель-Авив › Спектакли" (theatre). The event is on /ru/theater and not on /ru/festivals. Typing "Люмагика" in the search box shows no suggestions at all, while "Миядзаки" shows the matching concert at once. The page also declares a canonical address dated 12 November (/2026/11/12/…), while it shows "25 ноября".

**Why it matters:** This is the event the home page promotes as "Смотрят сейчас" (being viewed now), and a visitor who searches for it by name finds nothing.

**Steps:**
1. On /ru type "Люмагика" in "Найдите своё событие" (no Enter).
2. Compare with "Миядзаки".
3. Open /ru/event/il/tel-aviv/2026/11/25/light-festival-lumagica-sg3haMfwLL4yDeJsz22a12 and read the breadcrumb and link[rel=canonical].

**Suggested fix:** Map the festival tag to the Festivals category; include multi-day and ongoing events in the search index; use a canonical URL without a date or with the next occurrence.

**Fixed when:** The festival is under Festivals, typing "Люмагика" suggests it, and the canonical address does not carry a past date.

**Check:**
```json
{
 "url": "https://www.ogo.co.il/ru/event/il/tel-aviv/2026/11/25/light-festival-lumagica-sg3haMfwLL4yDeJsz22a12",
 "viewport": "desktop",
 "wait_ms": 2500,
 "assert_bug_present": [
  {
   "js_true": "document.querySelector('link[rel=canonical]').href.includes('/2026/11/12/')"
  },
  {
   "text_present": "Спектакли"
  }
 ]
}
```

## OBS-03 · Filters show events that do not match them  
**Priority:** should fix  
**Viewport:** desktop  
**Pages:** https://www.ogo.co.il/ru/concerts/filtered?tod=day, https://www.ogo.co.il/en/concerts/filtered?tod=day, https://www.ogo.co.il/en/kids/filtered?age=teens

**What we saw:** Under the daytime filter "днем, до 17:00" (daytime, before 17:00) the first card says "25 ноября в 18:00", and the next one "26 ноября в 19:00". The English list shows "6:00 PM" and "7:00 PM". The kids filter "Teens 13–17" lists "Guli and the Singing Guitar for Crawling Age to Two Years Old" and "The Maccabees … Ages 2–12".

**Why it matters:** A parent who picks "teens" or "daytime" has to check every card again by hand.

**Steps:**
1. Open /ru/concerts and choose "Днем".
2. Read the times on the first cards.
3. Open /en/kids and choose "Teens 13–17".

**Suggested fix:** When a filter matches a multi-session event on one session, show that session on the card. Match age filters on the event age range, not on a broad "kids" tag.

**Fixed when:** Under the daytime filter each card shows a session before 17:00, or the event is left out. The teens list has no events for toddlers.

**Check:**
```json
{
 "url": "https://www.ogo.co.il/ru/concerts/filtered?tod=day",
 "viewport": "desktop",
 "wait_ms": 3000,
 "assert_bug_present": [
  {
   "js_true": "/\\d+ [а-я]+ в (1[7-9]|2\\d):\\d\\d/.test(document.body.innerText)"
  }
 ]
}
```

## OBS-04 · Back skips the list after date chips and page numbers  
**Priority:** should fix  
**Viewport:** desktop and mobile  
**Pages:** https://www.ogo.co.il/ru/concerts, https://www.ogo.co.il/ru/concerts/filtered?page=2

**What we saw:** On /ru/concerts, tapping "Сегодня" (today) changes the address to …/filtered?start=…&end=… but does not add a history step. Back then goes to the page before the list, or out of the site if the list was the first page opened. "Выходные" (weekend) and "Страница 2" (page 2) do the same. The "Ещё" panel, the city picker and the language chips do add a step, and Back works there.

**Why it matters:** A visitor who came from a search result and presses Back after one tap leaves the site.

**Steps:**
1. Open /ru/concerts from another page.
2. Tap "Сегодня" (or "Выходные", or page 2).
3. Press Back.

**Suggested fix:** Use router.push instead of router.replace for the date chips and the pagination.

**Fixed when:** After "Сегодня" or page 2, Back returns to the list as it was.

**Check:**
```json
{
 "url": "https://www.ogo.co.il/ru/concerts",
 "viewport": "desktop",
 "wait_ms": 5000,
 "assert_bug_present": [
  {
   "js_true": "(async()=>{const n=history.length;[...document.querySelectorAll('button')].find(b=>b.innerText.trim()==='Сегодня').click();await new Promise(r=>setTimeout(r,6000));return history.length===n && location.href.includes('start=')})()"
  }
 ]
}
```

## OBS-05 · A weekly meeting appears as five separate cards  
**Priority:** should fix  
**Viewport:** desktop  
**Pages:** https://www.ogo.co.il/ru/city/haifa/free, https://www.ogo.co.il/ru/city/haifa/lectures, https://www.ogo.co.il/ru/city/bnei-brak, https://www.ogo.co.il/ru/city/jerusalem/free

**What we saw:** "Встреча «Шлувот»" (a meeting) appears 5 times on /ru/city/haifa/free (29 Nov, 6, 13, 20 and 27 Dec). "Парламент Рамат-Эшколь" appears 4 times on the Haifa lectures page, and "Тайная жизнь Бней-Брака" 4 times on the Bnei Brak page, with four different end dates. Other series on the site are grouped into one card with "Ещё N дат" (N more dates).

**Why it matters:** Repeats push other events off the first screen and make a short list look longer.

**Steps:**
1. Open /ru/city/haifa/free.
2. Count the "Встреча «Шлувот»" cards.

**Suggested fix:** Apply the existing series grouping to occurrences that share title, venue and time but have different ids.

**Fixed when:** Each series shows as one card with "Ещё N дат", as other series already do.

**Check:**
```json
{
 "url": "https://www.ogo.co.il/ru/city/haifa/free",
 "viewport": "desktop",
 "wait_ms": 2500,
 "assert_bug_present": [
  {
   "js_true": "(document.body.innerText.match(/Встреча «Шлувот»/g)||[]).length >= 3"
  }
 ]
}
```

## OBS-06 · Items in lists they do not belong to: a meeting in Online, a season ticket in Free  
**Priority:** should fix  
**Viewport:** desktop  
**Pages:** https://www.ogo.co.il/ru/online, https://www.ogo.co.il/ru/city/haifa/free, https://www.ogo.co.il/en/places?free=1

**What we saw:** "Онлайн-события" (online events) lists "Скоростные гей-свидания в Иерусалиме…" (speed dating in Jerusalem), tagged both "встреча" (meeting) and "онлайн". "Бесплатные события в Хайфе" (free events) lists "Хапоэль Хайфа — абонементы с благотворительным взносом" (season tickets with a donation), tagged "за донат". 6 donation items appear on free pages. In free places, the Eilat Camel Ranch card says "from ₪100"; its page says entry is free and only extras are paid.

**Why it matters:** A visitor who picks "online" or "free" gets things they would have to travel to or pay for.

**Steps:**
1. Open /ru/online and find "Скоростные гей-свидания".
2. Open /ru/city/haifa/free and find "абонементы".

**Suggested fix:** Drop the online tag when an event has a venue and no stream link; keep "за донат" and ticket products off /free or add a filter for them; show "Free" on free places.

**Fixed when:** Online lists only remote events, Free lists only free-entry items (donation items labelled or kept apart), and a free place shows "Free" on its card.

**Check:**
```json
{
 "url": "https://www.ogo.co.il/ru/online",
 "viewport": "desktop",
 "wait_ms": 2500,
 "assert_bug_present": [
  {
   "text_present": "Скоростные гей-свидания в Иерусалиме"
  }
 ]
}
```

## OBS-07 · The filter panel offers choices that give no results, and its badge undercounts  
**Priority:** minor  
**Viewport:** desktop  
**Pages:** https://www.ogo.co.il/ru/concerts, https://www.ogo.co.il/ru/city/haifa/concerts/filtered?cost_types=free

**What we saw:** In the "Ещё" (more) panel, choosing "Хайфа" gives "Показать 306 событий"; adding "бесплатно" (free) gives "Показать 0 событий" and the button still works, leading to "События не найдены". The selected price chip gets a ✓, the selected city only a ring. After applying both, the badge reads "Ещё · 1".

**Why it matters:** A visitor walks into an empty page without warning and cannot tell how many filters are on.

**Steps:**
1. Open /ru/concerts and click "Ещё".
2. Choose "Хайфа", then "бесплатно".
3. Read the button, then press it.

**Suggested fix:** Compute facet counts for the current selection; use one selected style; count only recognised filter values in the badge.

**Fixed when:** Choices that would give 0 results are marked or disabled, selected chips look the same in every group, and the badge counts every active filter.

**Check:**
```json
{
 "url": "https://www.ogo.co.il/ru/city/haifa/concerts/filtered?cost_types=free",
 "viewport": "desktop",
 "wait_ms": 8000,
 "assert_bug_present": [
  {
   "text_present": "События не найдены"
  }
 ]
}
```

## OBS-08 · The city picker finds only Russian spellings  
**Priority:** minor  
**Viewport:** desktop  
**Pages:** https://www.ogo.co.il/ru/concerts

**What we saw:** Typing "хай" finds "Хайфа". Typing "Haifa" or "חיפה" shows "Такого города нет в списке" (no such city in the list). The main search box does find Latin and Hebrew text.

**Why it matters:** Many visitors type a city in English or Hebrew, and the picker tells them the city does not exist.

**Steps:**
1. Open /ru/concerts.
2. Click "Израиле" to choose a city.
3. Type "Haifa", then "חיפה".

**Suggested fix:** Index the Russian, Hebrew and English names and the slug of each city in the picker filter.

**Fixed when:** "Haifa" and "חיפה" both find Хайфа in the picker.

**Check:** none automatic; re-test by hand with the steps above.

## OBS-09 · Some filters are lost on Back or reload  
**Priority:** minor  
**Viewport:** desktop and mobile  
**Pages:** https://www.ogo.co.il/ru/calendar, https://www.ogo.co.il/en/festivals

**What we saw:** On the calendar, clicking "Каникулы" (school breaks) shows 7 items but the address stays /ru/calendar. After a reload all 11 items show again with "Все" selected. On /en/festivals, choosing "Next month", opening a festival and pressing Back returns to "All".

**Why it matters:** A visitor who opens one item and comes back has to set the filter again, and cannot share a filtered link.

**Steps:**
1. Open /ru/calendar, click "Каникулы", reload.
2. Open /en/festivals, choose "Next month", open a festival, press Back.

**Suggested fix:** Store the tab and period in a query parameter and read it on load.

**Fixed when:** The chosen tab or period is in the address (for example ?type=school), so Back, reload and shared links keep it.

**Check:**
```json
{
 "url": "https://www.ogo.co.il/ru/calendar",
 "viewport": "desktop",
 "wait_ms": 5000,
 "assert_bug_present": [
  {
   "js_true": "(async()=>{[...document.querySelectorAll('button')].find(b=>b.innerText.includes('Каникулы')).click();await new Promise(r=>setTimeout(r,1500));return location.href==='https://www.ogo.co.il/ru/calendar'})()"
  }
 ]
}
```

## OBS-10 · Pages linked from the home page with only one or two events  
**Priority:** minor  
**Viewport:** desktop  
**Pages:** https://www.ogo.co.il/ru/city/ashkelon/food, https://www.ogo.co.il/ru/city/ashkelon/lectures, https://www.ogo.co.il/ru/city/ashkelon/free, https://www.ogo.co.il/ru/city/latrun/kids

**What we saw:** Of the 44 city and category pages linked from the Russian home page, 4 have 2 items or fewer. "Еда в Ашкелоне" (food in Ashkelon) has 1 item, a crafts fair ("Ярмарка ручной работы"). "Лекции в Ашкелоне" has 1, "Бесплатные события в Ашкелоне" 2, "Детские события в Латруне" 2. The other 40 have 11 to 60.

**Why it matters:** A visitor follows a promoted link and lands on an almost empty page.

**Steps:**
1. Open /ru.
2. Click the Ashkelon "Еда" link.

**Suggested fix:** Link a city/category page only when it has 3 or more upcoming items; otherwise link the city page or the national category.

**Fixed when:** A city or category page is linked from the home page only when it has at least 3 upcoming items.

**Check:**
```json
{
 "url": "https://www.ogo.co.il/ru/city/ashkelon/food",
 "viewport": "desktop",
 "wait_ms": 2500,
 "assert_bug_present": [
  {
   "js_true": "new Set([...document.querySelectorAll('a[href*=\"/event/\"]')].map(a=>a.getAttribute('href'))).size <= 2"
  }
 ]
}
```

## OBS-11 · 20 events have no category, so no category page lists them  
**Priority:** minor  
**Viewport:** desktop  
**Pages:** https://www.ogo.co.il/ru/city/merkaz-meirav, https://www.ogo.co.il/ru/city/yifat, https://www.ogo.co.il/ru/city/bnei-brak

**What we saw:** 20 of 2,229 cards show only a price and a language, no category. Examples: "Спектакль «Зарезервированные места»" (a play), "Альта Виа 1", "«Бней-Браки́т»: аутентичная гастрономическая экскурсия" (a food tour), "Души".

**Why it matters:** A visitor browsing Theatre or Tours never sees these events.

**Steps:**
1. Open /ru/city/merkaz-meirav and scroll to June 2027.

**Suggested fix:** Require one category at import, inferred from the title ("Спектакль" → спектакль, "экскурсия" → экскурсия).

**Fixed when:** Every event card shows at least one category.

**Check:**
```json
{
 "url": "https://www.ogo.co.il/ru/city/yifat",
 "viewport": "desktop",
 "wait_ms": 2500,
 "assert_bug_present": [
  {
   "js_true": "document.body.innerText.includes('Альта Виа 1\\nЙифат\\nплатно')"
  }
 ]
}
```

## OBS-12 · A places search says "1" and "Nothing for this choice" at the same time  
**Priority:** minor  
**Viewport:** desktop  
**Pages:** https://www.ogo.co.il/en/places?type=beach&q=Atlit

**What we saw:** The filters read "All · 1" and "Museums & heritage · 1", and the list below says "Nothing for this choice."

**Why it matters:** The visitor sees there is a match but cannot reach it.

**Steps:**
1. Open /en/places?type=beach&q=Atlit.
2. Compare the filter counts with the list.

**Suggested fix:** Compute the counts with the same type filter as the list, or show the match when the selected type has none.

**Fixed when:** The counts and the list agree: either the one place is shown, or the counts say 0.

**Check:**
```json
{
 "url": "https://www.ogo.co.il/en/places?type=beach&q=Atlit",
 "viewport": "desktop",
 "wait_ms": 3500,
 "assert_bug_present": [
  {
   "text_present": "All · 1"
  },
  {
   "text_present": "Nothing for this choice"
  }
 ]
}
```

## OBS-13 · A tour card says "43 more dates until 19 Nov"; the tour page says 40 dates until May 2027  
**Priority:** fix first  
**Viewport:** desktop and mobile  
**Pages:** https://www.ogo.co.il/ru, https://www.ogo.co.il/he, https://www.ogo.co.il/ru/event/il/ashdod/2026/11/27/gior-singer-9UCX1A4foNb7ppRHAohqC6

**What we saw:** The home card for "Стендап Гиоры Зингера" (Friday 27 Nov) says "Ещё 43 даты до 19 нояб." (43 more dates until 19 Nov) and "…и ещё 30 городов" (30 more cities). The event page says "Гастроли: 40 дат в 32 городах, до 29 мая 2027 г." (40 dates in 32 cities, until 29 May 2027). The Hebrew home card says the same as the Russian one. We checked 7 other tour cards and they match their pages.

**Why it matters:** "until 19 Nov" on a 27 Nov card reads as a tour that is already over, so a visitor may not look for a later date.

**Steps:**
1. Open /ru and find the Gior Singer card under Friday 27 November.
2. Open it and compare with "Гастроли: …".

**Suggested fix:** Build the card's 'N more dates / until X / +K cities' from the same deduplicated, future-only date list the event page uses; print the year when it differs.

**Fixed when:** The card shows the same count and end date as the tour page, with the year when the end date is in another year ("Ещё 39 дат до 29 мая 2027").

**Check:**
```json
{
 "url": "https://www.ogo.co.il/ru",
 "viewport": "desktop",
 "wait_ms": 2500,
 "assert_bug_present": [
  {
   "text_present": "Ещё 43 даты до 19 нояб."
  }
 ]
}
```

## OBS-14 · The Jerusalem Hanukkah page says to light the first candle after sunset on a Friday  
**Priority:** fix first  
**Viewport:** desktop  
**Pages:** https://www.ogo.co.il/en/holidays/hanukkah/jerusalem

**What we saw:** "First candle, December 4 | 4:34 PM | after sunset". 4 December 2026 is a Friday. On Friday the Hanukkah candles are lit before the Shabbat candles, so before sunset; in Jerusalem Shabbat candles are lit about 40 minutes before sunset.

**Why it matters:** An observant visitor who follows the page would light too late on the first night.

**Steps:**
1. Open /en/holidays/hanukkah/jerusalem.
2. Read the first-candle line.

**Suggested fix:** When the first night falls on a Friday, show the time before Shabbat candle lighting.

**Fixed when:** For Friday 4 December the page gives a time before Shabbat candle lighting, or says "before Shabbat candles".

**Check:**
```json
{
 "url": "https://www.ogo.co.il/en/holidays/hanukkah/jerusalem",
 "viewport": "desktop",
 "wait_ms": 3000,
 "assert_bug_present": [
  {
   "text_present": "First candle, December 4"
  },
  {
   "text_present": "after sunset"
  }
 ]
}
```

## OBS-15 · Self-guided phone tours appear on Today with made-up start times  
**Priority:** fix first  
**Viewport:** desktop  
**Pages:** https://www.ogo.co.il/ru/today, https://www.ogo.co.il/ru/city/jerusalem/free, https://www.ogo.co.il/ru/city/tel-aviv

**What we saw:** "25 ноября в 08:00 · Идеальное свидание: самостоятельная ночная экскурсия по Иерусалиму с мобильным телефоном" (a self-guided night tour, at 08:00). Others show "ежедневно в 08:00", "в 16:00", "в 11:45" (daily at a fixed time). These tours can be done at any time and belong on /ru/anyday ("Дату выбираете сами", you choose the date).

**Why it matters:** A visitor reads a fixed start time that does not exist, and a night tour at 8 in the morning.

**Steps:**
1. Open /ru/today and look for "ночная экскурсия".

**Suggested fix:** Classify app-based self-guided tours as anyday at import and keep them off day lists.

**Fixed when:** Self-guided tours appear only under "В любой день" (any day) and show no clock time.

**Check:**
```json
{
 "url": "https://www.ogo.co.il/ru/today",
 "viewport": "desktop",
 "wait_ms": 2500,
 "assert_bug_present": [
  {
   "text_present": "самостоятельная ночная экскурсия"
  }
 ]
}
```

## OBS-16 · The same show is listed twice, and the copies disagree on price, time or box office  
**Priority:** fix first  
**Viewport:** desktop  
**Pages:** https://www.ogo.co.il/ru/exhibitions, https://www.ogo.co.il/en/event/il/haifa/2026/06/28/exhibition-in-praise-shadows-or-silence-light-1q5cbcDimJtqvx1XyJdb6j, https://www.ogo.co.il/en/event/il/haifa/2026/06/27/in-praise-shadows-or-silence-time-light-3wXPeEvH1XkfAKtJ19JtJN, https://www.ogo.co.il/ru/city/beer-sheva/concerts, https://www.ogo.co.il/en/city/haifa/weekend

**What we saw:** The In Praise of Shadows exhibition in Haifa is listed twice, both "До 28 ноября": "Выставка «Восхваление теней | Тишина света»" tagged "бесплатно" (free) and "Во славу теней | Безмолвие времени света" tagged "платно" (paid). In English one page has "Register", the other "Buy Tickets". The Streisand tribute in Beer-Sheva on 28 Nov is listed twice: one page names the Kivunim box office, the other a Tel Aviv hall and phone number. Nino Katamadze in Haifa on 27 Nov is listed at 8:00 PM and at 8:30 PM. 6 such pairs in all.

**Why it matters:** A visitor does not know whether to pay, when to arrive, or whom to call.

**Steps:**
1. Open /ru/exhibitions and find both In Praise of Shadows cards under "До 28 ноября".
2. Open /en/city/haifa/weekend and find both Nino Katamadze cards.

**Suggested fix:** Deduplicate at import by city, venue, start time and normalised title (or translation of the source title); keep the primary ticket seller.

**Fixed when:** One card per show, with one price, one time and one ticket seller.

**Check:**
```json
{
 "url": "https://www.ogo.co.il/ru/exhibitions",
 "viewport": "desktop",
 "wait_ms": 2500,
 "assert_bug_present": [
  {
   "text_present": "Восхваление теней"
  },
  {
   "text_present": "Во славу теней"
  }
 ]
}
```

## OBS-17 · Events and places are filed under the wrong town, one of them in Italy  
**Priority:** fix first  
**Viewport:** desktop  
**Pages:** https://www.ogo.co.il/ru/city/yifat, https://www.ogo.co.il/en/event/il/tel-aviv/2026/11/25/cinem-club-many-faces-series-5ft61YfyVif9DQZH1X1LB2, https://www.ogo.co.il/en/places?type=forest&city=tel-aviv&free=1, https://www.ogo.co.il/ru/city/kfar-truman

**What we saw:** The Yifat page lists "Альта Виа 1", 3–10 August 2027. Its page describes a mountain trek in the Dolomites in northern Italy. The Cinema Club page gives "Tel Aviv-Yafo" and directions to "ANU – Museum of the Jewish People", while its summary says the meetings are "at Mishkan in Ra'anana". "Forests in Tel Aviv" shows forests in Oranit and Shoham.

**Why it matters:** A visitor may set off for the wrong town, or think there is a trek near Yifat.

**Steps:**
1. Open /ru/city/yifat, scroll to August 2027, open "Альта Виа 1".
2. Open /en/event/il/tel-aviv/2026/11/25/cinem-club-many-faces-series-5ft61YfyVif9DQZH1X1LB2 and compare the destination with the summary.

**Suggested fix:** Resolve the city from the venue's coordinates, not the organiser's address; flag events whose description names a foreign country.

**Fixed when:** Events are filed under the town where they take place, events abroad are kept off Israeli city pages, and the map points to the real venue.

**Check:**
```json
{
 "url": "https://www.ogo.co.il/ru/city/yifat",
 "viewport": "desktop",
 "wait_ms": 2500,
 "assert_bug_present": [
  {
   "text_present": "Альта Виа 1"
  }
 ]
}
```

## OBS-18 · Event summaries contradict the date, time or place on the same page  
**Priority:** fix first  
**Viewport:** desktop  
**Pages:** https://www.ogo.co.il/en/event/il/haifa/2026/06/28/exhibition-in-praise-shadows-or-silence-light-1q5cbcDimJtqvx1XyJdb6j, https://www.ogo.co.il/en/event/il/jerusalem/2026/11/25/concert-hysteria-9xDP6dPgF9X8XR3wSwUAJQ, https://www.ogo.co.il/en/event/il/jerusalem/2026/11/25/fair-talpiot-food-market-vE45wUs9VjQJYYXq4WtMAX, https://www.ogo.co.il/en/event/il/beer-sheva/2026/11/25/chefs-games-community-wednesday-yes-community-initiative-third-ge-6uCQou6A478ipojAgKPJa5, https://www.ogo.co.il/en/event/il/ramat-gan/2026/11/25/lectures-by-ramat-hen-veterans-club-f-month-november-iyfA1yJEmSVnnbxsuUk5Pf

**What we saw:** In Praise of Shadows: the summary says "It runs from 28 November 2026 to 28 June 2026" (backwards); the schedule says "June 28 – November 28". Hysteria: the summary says "November 25, 26, and 28, 2026, at Pais Arena"; the description says Menora Hall and Jerusalem Arena during Hanukkah. Talpiot Food Market: the location says "Ben Yehuda School, Nakhshon St 11"; the summary says "Nivaya School, 6 Kibbutz Galuyot St.". Chef's Games: card 4:30 PM, summary "16:00–18:30". Ramat Hen: schedule "Wednesdays and Sundays at 11:30", summary "Sundays at 11:30 and Wednesdays at 18:00".

**Why it matters:** A visitor reads the summary first and may go to the wrong place or at the wrong time.

**Steps:**
1. Open each page.
2. Compare the SUMMARY block with the schedule and the location block.

**Suggested fix:** Generate the summary from the structured date, time and venue fields, or check it against them and flag mismatches for review.

**Fixed when:** On each of these pages the summary gives the same dates, times and venue as the schedule and location block.

**Check:**
```json
{
 "url": "https://www.ogo.co.il/en/event/il/haifa/2026/06/28/exhibition-in-praise-shadows-or-silence-light-1q5cbcDimJtqvx1XyJdb6j",
 "viewport": "desktop",
 "wait_ms": 3000,
 "assert_bug_present": [
  {
   "text_present": "from 28 November 2026 to 28 June 2026"
  }
 ]
}
```

## OBS-19 · Tour cards sit on a later date and hide an earlier one  
**Priority:** should fix  
**Viewport:** desktop  
**Pages:** https://www.ogo.co.il/ru

**What we saw:** "Концерт REFLEX" is listed under Friday 27 Nov with "Ещё 1 дата до 27 нояб." (1 more date until 27 Nov). The other date is 26 Nov in Tel Aviv, a day earlier. The same happens for Sasha Kroyter (listed 10 Dec, tour from 1 Dec), Tanya Vinokur (listed 4 Dec, other dates 25 and 26 Nov), Shaov and Gissen.

**Why it matters:** A visitor who reads "until 27 Nov" on a 27 Nov card thinks this is the only chance and misses the earlier show.

**Steps:**
1. Open /ru, find "Концерт REFLEX" under Friday 27 November.
2. Open it, click "Все даты и города": 26 Nov Tel Aviv, 27 Nov Haifa.

**Suggested fix:** For tour cards, list the next upcoming instance or show the from–to range.

**Fixed when:** A tour card sits at its next date, or says where the other dates are ("ещё 1 дата: 26 нояб., Тель-Авив").

**Check:**
```json
{
 "url": "https://www.ogo.co.il/ru",
 "viewport": "desktop",
 "wait_ms": 2500,
 "assert_bug_present": [
  {
   "text_present": "Ещё 1 дата до 27 нояб."
  }
 ]
}
```

## OBS-20 · The same holiday has different event counts on the home page, the holidays page and the calendar  
**Priority:** should fix  
**Viewport:** desktop  
**Pages:** https://www.ogo.co.il/en, https://www.ogo.co.il/en/calendar, https://www.ogo.co.il/ru, https://www.ogo.co.il/ru/holidays, https://www.ogo.co.il/ru/calendar, https://www.ogo.co.il/en/holidays, https://www.ogo.co.il/ru/exhibitions/filtered?closing=1

**What we saw:** Hanukkah (4–12 Dec): "1,174 events" on the English and Hebrew home pages, "1 193 события" on the Russian home and holidays pages, "1,221 events" on the calendar in all three languages. New Year: 148 (Russian holidays page), 82 (English), 172 (calendar). Purim: 51 on the holidays page, 40 on the calendar. Exhibitions closing soon: the home page says "22 выставки", the page it links to lists 21, the calendar says "36 событий".

**Why it matters:** A visitor sees three numbers for one thing and stops trusting any of them.

**Steps:**
1. Open /en and read the "Holiday soon" tile.
2. Open /en/calendar and read the Hanukkah card.
3. Open /ru, /ru/holidays and /ru/exhibitions/filtered?closing=1 and compare.

**Suggested fix:** Compute holiday and collection counts in one function (same date window, language filter and deduplication) and use it for the home tile, /holidays, /calendar and city pages.

**Fixed when:** One holiday or collection has one number everywhere, and the teaser count equals the number of items on the page it links to.

**Check:**
```json
{
 "url": "https://www.ogo.co.il/en/calendar",
 "viewport": "desktop",
 "wait_ms": 2500,
 "assert_bug_present": [
  {
   "text_present": "1,221 events"
  },
  {
   "js_true": "fetch('/en').then(r=>r.text()).then(t=>{const m=t.replace(/<[^>]+>/g,' ').match(/Hanukkah[\\s\\S]{0,160}?([\\d,]+)\\s*events/);return !!m && m[1]!=='1,221'})"
  }
 ]
}
```

## OBS-21 · City pages give two different event counts on the same page  
**Priority:** should fix  
**Viewport:** desktop  
**Pages:** https://www.ogo.co.il/ru/city/ashdod/today, https://www.ogo.co.il/ru/city/haifa/today, https://www.ogo.co.il/ru/city/tel-aviv/today, https://www.ogo.co.il/ru/city/jerusalem/today, https://www.ogo.co.il/ru/city/ashkelon/today, https://www.ogo.co.il/en/city/haifa, https://www.ogo.co.il/en/city/haifa/weekend

**What we saw:** On "Сегодня в Ашдоде" the browser tab says "9 событий", the answer below says "Сегодня в Ашдоде — 15 событий" (15 events today) and the list shows 5 events. Haifa: 45 and 95. Tel Aviv: 270 and 443. Jerusalem: 136 and 168. Ashkelon: 4 and 5. On the English Haifa page the weekend link says "All 54", the weekend page says "120 events are planned", and its first page lists 60.

**Why it matters:** A visitor planning a day cannot tell how much is on.

**Steps:**
1. Open /ru/city/ashdod/today.
2. Compare the tab title, the answer to "Что происходит в Ашдоде сегодня?" and the cards.

**Suggested fix:** Compute title, FAQ and list from one query with the same filters; build the lead sentence from the listed items.

**Fixed when:** The tab title, the answer text and the list give one number, or the text says what the larger number includes.

**Check:**
```json
{
 "url": "https://www.ogo.co.il/ru/city/ashdod/today",
 "viewport": "desktop",
 "wait_ms": 3000,
 "assert_bug_present": [
  {
   "js_true": "(()=>{const a=(document.title.match(/— (\\d+) событ/)||[])[1]; const b=(document.body.innerText.match(/Сегодня в Ашдоде — (\\d+) событ/)||[])[1]; return a&&b&&a!==b})()"
  }
 ]
}
```

## OBS-22 · Hanukkah pages show date headings out of order, one of them twice  
**Priority:** should fix  
**Viewport:** desktop and mobile  
**Pages:** https://www.ogo.co.il/ru/holidays/hanukkah, https://www.ogo.co.il/en/holidays/hanukkah, https://www.ogo.co.il/he/holidays/hanukkah, https://www.ogo.co.il/ru/holidays/hanukkah/haifa

**What we saw:** The headings run "2–8 декабря", then "Вторник, 1 декабря" (Tuesday 1 December), then "2–8 декабря" again. The first "2–8 декабря" group holds the 4 December events. The English and Hebrew pages and the Haifa page do the same.

**Why it matters:** A visitor scanning by date for a Hanukkah night sees the same week twice and a date that goes backwards.

**Steps:**
1. Open /ru/holidays/hanukkah.
2. Read the date headings above the cards.

**Suggested fix:** Sort groups by date before rendering; put events that started before the holiday in one "весь праздник" group.

**Fixed when:** Date headings run in order and each appears once; events that began before the holiday sit in one "all holiday" group.

**Check:**
```json
{
 "url": "https://www.ogo.co.il/ru/holidays/hanukkah",
 "viewport": "desktop",
 "wait_ms": 2500,
 "assert_bug_present": [
  {
   "js_true": "[...document.querySelectorAll('h2')].map(x=>x.innerText.trim()).filter(x=>x==='2–8 декабря').length>1"
  }
 ]
}
```

## OBS-23 · "Shushan Purim (Jerusalem)" opens the general Purim page  
**Priority:** should fix  
**Viewport:** desktop  
**Pages:** https://www.ogo.co.il/en/calendar, https://www.ogo.co.il/en/holidays/purim

**What we saw:** On the calendar, "Purim" and "Shushan Purim (Jerusalem)" both link to /en/holidays/purim. That page covers 22–24 March, lists Tel Aviv events first and has no Jerusalem filter.

**Why it matters:** A Jerusalem visitor looking for Shushan Purim events gets the national list.

**Steps:**
1. Open /en/calendar.
2. Compare the links of "Purim" and "Shushan Purim (Jerusalem)".

**Suggested fix:** Link Shushan Purim to /en/holidays/purim/jerusalem (or a filtered view for its date).

**Fixed when:** "Shushan Purim (Jerusalem)" opens Jerusalem events on Shushan Purim.

**Check:**
```json
{
 "url": "https://www.ogo.co.il/en/calendar",
 "viewport": "desktop",
 "wait_ms": 3000,
 "assert_bug_present": [
  {
   "js_true": "(()=>{const h=[...document.querySelectorAll('a')].filter(a=>/^(Purim|Shushan Purim \\(Jerusalem\\))$/.test(a.innerText.trim())).map(a=>a.getAttribute('href'));return h.length>=2 && new Set(h).size===1})()"
  }
 ]
}
```

## OBS-24 · A cinema shows "Open until 18:00" in the middle of the night  
**Priority:** minor  
**Viewport:** desktop  
**Pages:** https://www.ogo.co.il/ru/places

**What we saw:** The first card, "Кинотеатр Яффа", says "Открыто до 18:00" (open until 18:00) at 01:10, 01:33 and 02:37 Israel time. Every other card at that hour says "Откроется в 09:00" (opens at 09:00).

**Why it matters:** A visitor may set off for a place that is closed.

**Steps:**
1. Open /ru/places at night, Israel time.

**Suggested fix:** Check the open-now logic for hours that cross midnight or have no opening time; compute it in Asia/Jerusalem.

**Fixed when:** At night the card says "Закрыто · откроется в …" (closed, opens at …).

**Check:**
```json
{
 "url": "https://www.ogo.co.il/ru/places",
 "viewport": "desktop",
 "wait_ms": 2500,
 "note": "only meaningful between 00:00 and 08:00 Israel time",
 "assert_bug_present": [
  {
   "js_true": "(()=>{const h=+new Date().toLocaleString('en-US',{timeZone:'Asia/Jerusalem',hour:'numeric',hour12:false});return h<8 && document.body.innerText.includes('Кинотеатр Яффа\\nОткрыто до 18:00')})()"
  }
 ]
}
```

## OBS-25 · The calendar promises "a year ahead" but stops at July 2027  
**Priority:** minor  
**Viewport:** desktop  
**Pages:** https://www.ogo.co.il/en/calendar

**What we saw:** The intro says the calendar covers "a year ahead". The month sections run from November 2026 to July 2027. Rosh Hashanah, Yom Kippur and Sukkot 2027 are not there.

**Why it matters:** A visitor planning the autumn holidays finds nothing and may think nothing is planned.

**Steps:**
1. Open /en/calendar.
2. Scroll to the last month section.

**Suggested fix:** Extend the calendar data to 12 months, or change the copy.

**Fixed when:** The calendar reaches November 2027, or the intro names the months it covers.

**Check:**
```json
{
 "url": "https://www.ogo.co.il/en/calendar",
 "viewport": "desktop",
 "wait_ms": 3000,
 "assert_bug_present": [
  {
   "text_present": "a year ahead"
  },
  {
   "js_true": "!/(^|\\n)(August|September|October) 2027(\\n|$)/.test(document.body.innerText)"
  }
 ]
}
```

## OBS-26 · A sold-out event offers "other dates" that lead back to the same sold-out page  
**Priority:** fix first  
**Viewport:** desktop  
**Pages:** https://www.ogo.co.il/ru/event/il/ashdod/2026/11/25/ashdods-70th-birthday-visit-center-crTCvm4TZwjht7dqeVdTrJ

**What we saw:** The page says "Билеты распроданы" (sold out) and offers "Есть билеты на другие даты →" (tickets for other dates). The link opens the same sold-out page. "Все даты" (all dates) lists "25 ноября · распродано", "25 ноября", "25 ноября". The page also says "Проходит по средам до 25 нояб." (Wednesdays until 25 Nov), so no later date exists.

**Why it matters:** A visitor looking for tickets goes round in a circle.

**Steps:**
1. Open the page.
2. Click "Есть билеты на другие даты →".
3. Open "Все даты ▾".

**Suggested fix:** Deduplicate sibling occurrences after the merge; hide the "other dates" link when the only alternatives are the same occurrence.

**Fixed when:** "other dates" appears only when another date has tickets, and each date is listed once.

**Check:**
```json
{
 "url": "https://www.ogo.co.il/ru/event/il/ashdod/2026/11/25/ashdods-70th-birthday-visit-center-crTCvm4TZwjht7dqeVdTrJ",
 "viewport": "desktop",
 "wait_ms": 2500,
 "assert_bug_present": [
  {
   "text_present": "Есть билеты на другие даты"
  },
  {
   "text_present": "Билеты распроданы"
  }
 ]
}
```

## OBS-27 · A postponed event: the title stops at "postponed to" and the page still shows the old date  
**Priority:** fix first  
**Viewport:** desktop  
**Pages:** https://www.ogo.co.il/ru/event/il/ein-hod/2026/12/04/quest-ur-rtists-village-postponed-bg5EQVChNhK4LGVwDrCGNq, https://www.ogo.co.il/he/event/il/ein-hod/2026/12/04/quest-ur-rtists-village-postponed-bg5EQVChNhK4LGVwDrCGNq

**What we saw:** The title reads "Экскурсия-квест по деревне художников. Перенос на" ("… Postponed to", with nothing after it). The Hebrew title stops at "נדחה ל-". The date block still says "4 декабря, 10:00–14:00", "Купить билеты" (buy tickets) is shown, and the page data still marks the event as scheduled.

**Why it matters:** A visitor can buy a ticket for a date that is no longer valid.

**Steps:**
1. Open the page in Russian and Hebrew.
2. Read the title and the date block.

**Suggested fix:** Keep the new date from the source title; parse "перенос на <date>" into a status and date; set eventStatus to EventPostponed or EventRescheduled.

**Fixed when:** The page shows "Перенесено" (postponed) with the new date, or no date and no ticket button until there is one.

**Check:**
```json
{
 "url": "https://www.ogo.co.il/ru/event/il/ein-hod/2026/12/04/quest-ur-rtists-village-postponed-bg5EQVChNhK4LGVwDrCGNq",
 "viewport": "desktop",
 "wait_ms": 2500,
 "assert_bug_present": [
  {
   "js_true": "/Перенос на$/.test(document.querySelector('h1').innerText.trim())"
  }
 ]
}
```

## OBS-28 · Price ranges read "from 126-286 ₪", and the hidden page data gives the top price  
**Priority:** should fix  
**Viewport:** desktop  
**Pages:** https://www.ogo.co.il/ru/event/il/ashdod/2026/11/25/miyazakis-world-music-performed-by-symphonians-5wq4HyCADgX9L5ikmWdqts, https://www.ogo.co.il/en/event/il/ashdod/2026/11/25/miyazakis-world-music-performed-by-symphonians-5wq4HyCADgX9L5ikmWdqts, https://www.ogo.co.il/he/event/il/ashdod/2026/11/25/miyazakis-world-music-performed-by-symphonians-5wq4HyCADgX9L5ikmWdqts, https://www.ogo.co.il/ru/event/il/tel-aviv/2026/11/26/vasily-petrenko-conduct-or-nn-shelest-pianist-sTkfTcG36sfXYbiRqmqYS4

**What we saw:** The page title in all three languages says "билеты от 126-286 ₪" / "tickets from 126-286 ₪". The price block shows "126-286 ₪". The data that search engines read gives a single price of 286. The Petrenko concert reads "от 220-640 ₪" with 640 in the data.

**Why it matters:** In search results the event can show the highest price as its price.

**Steps:**
1. Open the page; read the browser tab title.
2. Read script[type="application/ld+json"] offers.price.

**Suggested fix:** When the price is a range, render "from {min}" and emit AggregateOffer with lowPrice and highPrice.

**Fixed when:** The title says "от 126 ₪" (or "126–286 ₪" without "от"), and the page data gives low price 126 and high price 286.

**Check:**
```json
{
 "url": "https://www.ogo.co.il/ru/event/il/ashdod/2026/11/25/miyazakis-world-music-performed-by-symphonians-5wq4HyCADgX9L5ikmWdqts",
 "viewport": "desktop",
 "wait_ms": 2000,
 "assert_bug_present": [
  {
   "js_true": "/от \\d+-\\d+ ₪/.test(document.title)"
  }
 ]
}
```

## OBS-29 · Deal cards under "Digitel" do not show the deal  
**Priority:** minor  
**Viewport:** desktop  
**Pages:** https://www.ogo.co.il/en/deals

**What we saw:** With the "Digitel" filter, the last-minute cards (Streamlining Processes, Treasure Beneath the Bridge, Pinocchio) show no provider and no discount. Without the filter, cards show "Hulyo | Ticket for 73 ₪", and "With cards and clubs" cards show "Digitel | 10 ₪ off".

**Why it matters:** On a deals page, a card without the deal gives the visitor no reason to click.

**Steps:**
1. Open /en/deals.
2. Choose "Digitel".
3. Read the last-minute cards.

**Suggested fix:** Render the provider and discount line for Digitel last-minute offers as for other providers.

**Fixed when:** Every card under "Digitel" shows the provider and the discount.

**Check:** none automatic; re-test by hand with the steps above.

## OBS-30 · Venue names and addresses appear in the wrong language or with several spellings  
**Priority:** should fix  
**Viewport:** desktop and mobile  
**Pages:** https://www.ogo.co.il/ru/event/il/ashdod/2026/11/25/miyazakis-world-music-performed-by-symphonians-5wq4HyCADgX9L5ikmWdqts, https://www.ogo.co.il/ru/event/il/tel-aviv/2026/11/26/vasily-petrenko-conduct-or-nn-shelest-pianist-sTkfTcG36sfXYbiRqmqYS4, https://www.ogo.co.il/he/event/il/tel-aviv/2026/11/26/vasily-petrenko-conduct-or-nn-shelest-pianist-sTkfTcG36sfXYbiRqmqYS4, https://www.ogo.co.il/he/city/ein-hod

**What we saw:** One venue is spelled "Леванваним Ашдод" in the venue line, "в Яд ле-Баним" in the summary on the same page and "Яд ла-Баним" on the Ashdod page. The address line "תחנה מרכזית, Sderot Menachem Begin 1, Ashdod" mixes Hebrew and English on the Russian page. The Petrenko concert gives only "Tel Aviv-Yafo, Israel" in English on Russian and Hebrew pages, and on a phone the ticket bar cuts it to "Tel Aviv-Yafo, Isr…". The Hebrew name of Ein Hod is spelled "עין חוד" (title and city page) and "עין הוד" (summary).

**Why it matters:** A visitor cannot match the venue on the page with the name on the map or the ticket.

**Steps:**
1. Open the Miyazaki concert in Russian; compare the venue line with the summary.
2. Open the Petrenko concert in Russian on a phone.

**Suggested fix:** Keep a venue dictionary with names per language; fall back to the venue name, not the city; fix the Hebrew spelling of Ein Hod.

**Fixed when:** Each venue has one name per language, addresses are in the page language, and a missing venue name is not replaced by the city.

**Check:**
```json
{
 "url": "https://www.ogo.co.il/ru/event/il/tel-aviv/2026/11/26/vasily-petrenko-conduct-or-nn-shelest-pianist-sTkfTcG36sfXYbiRqmqYS4",
 "viewport": "mobile",
 "wait_ms": 3500,
 "assert_bug_present": [
  {
   "text_present": "Tel Aviv-Yafo, Israel"
  }
 ]
}
```

## OBS-31 · Russian text on English pages, Hebrew titles on Russian pages, and broken text  
**Priority:** should fix  
**Viewport:** desktop  
**Pages:** https://www.ogo.co.il/en/event/il/tel-aviv/2026/11/25/design-week-israel-ike7HGCGbDSzEsKmNo6dAb, https://www.ogo.co.il/ru/festivals, https://www.ogo.co.il/ru/city/jerusalem/concerts, https://www.ogo.co.il/ru/city/haifa/free, https://www.ogo.co.il/en/event/il/tel-aviv/2026/11/12/exhibition-opening-rchitects-dream-or-maj-gratzfeld-and-roy-zvulun-kqZxi1ePt8REMaKV4JBCuB

**What we saw:** The English Design Week page has its summary in Russian: "Design Week Tel Aviv — четыре дня выставок…". /ru/festivals shows a festival called "200 שנות צילום" in Hebrew. Other Russian titles mix alphabets ("Сиhара"), repeat themselves ("Книжный клуб 2026: … | Книжный клуб 2026: …") or carry the schedule and street inside the title. The Architect's Dream description stops mid-sentence: "…and between the".

**Why it matters:** A visitor reads a page they cannot understand, or text that looks broken.

**Steps:**
1. Open the Design Week page in English and read SUMMARY.
2. Open /ru/festivals and scroll to 17–19 December.

**Suggested fix:** Add a check at import: no Hebrew letters in Russian titles, no Cyrillic in English text, no repeated "A | A", no time or street inside titles; send failures back for translation.

**Fixed when:** Russian pages have no Hebrew-only titles, English pages have no Russian text, and descriptions end in full sentences.

**Check:**
```json
{
 "url": "https://www.ogo.co.il/ru/festivals",
 "viewport": "desktop",
 "wait_ms": 2500,
 "assert_bug_present": [
  {
   "text_present": "200 שנות צילום"
  }
 ]
}
```

## OBS-32 · English and Hebrew pages keep the raw promotional title  
**Priority:** minor  
**Viewport:** desktop  
**Pages:** https://www.ogo.co.il/en/event/il/haifa/2026/11/25/performance-by-zheny-berkovich-tragicomedy-our-treasure-performance-has-been-h445tvMFxLBRRD8R46tJwd, https://www.ogo.co.il/he/event/il/haifa/2026/11/25/performance-by-zheny-berkovich-tragicomedy-our-treasure-performance-has-been-h445tvMFxLBRRD8R46tJwd, https://www.ogo.co.il/ru/event/il/haifa/2026/11/25/performance-by-zheny-berkovich-tragicomedy-our-treasure-performance-has-been-h445tvMFxLBRRD8R46tJwd

**What we saw:** Russian: "Спектакль Жени Беркович «Наше сокровище»". English: "Performance by Zhenya Berkovich - Tragicomedy Our Treasure - the performance has been removed from the theater's repertoire in Russia". The Hebrew title is the same long line. The Petrenko concert is "Концерт Василия Петренко и Анны Шелест" in Russian and "Vasily Petrenko conductor | Anna Shelest pianist" in English.

**Why it matters:** Visitors who switch language think they are looking at a different event.

**Steps:**
1. Open the event in Russian, then switch to English and Hebrew.
2. Compare the titles.

**Suggested fix:** Translate from the cleaned title field, not the raw source title.

**Fixed when:** The English and Hebrew titles are translations of the cleaned Russian title.

**Check:**
```json
{
 "url": "https://www.ogo.co.il/en/event/il/haifa/2026/11/25/performance-by-zheny-berkovich-tragicomedy-our-treasure-performance-has-been-h445tvMFxLBRRD8R46tJwd",
 "viewport": "desktop",
 "wait_ms": 2000,
 "assert_bug_present": [
  {
   "text_present": "removed from the theater's repertoire"
  }
 ]
}
```

## OBS-33 · The city weather row runs backwards in Hebrew and is cut off on small phones  
**Priority:** should fix  
**Viewport:** mobile  
**Pages:** https://www.ogo.co.il/he/city/tel-aviv, https://www.ogo.co.il/ru/city/tel-aviv, https://www.ogo.co.il/en/city/tel-aviv

**What we saw:** In Hebrew the three days read Friday, Thursday, Wednesday from the right, where a Hebrew reader starts, so today is last. Each Hebrew day label breaks onto two lines ("יום" / "ד׳"), and at 320 px today's label is pushed off the left edge. In Russian and English at 320 px the row is wider than the screen and the last day is cut off ("Пт ☀ 2…").

**Why it matters:** A visitor checking the weather for a day out reads the wrong day or cannot see it.

**Steps:**
1. Open /he/city/tel-aviv on a phone.
2. Open /ru/city/tel-aviv at 320 px wide.

**Suggested fix:** Remove the fixed dir=ltr on the weather row (keep it only on the temperature numbers), add white-space:nowrap to day labels, and reduce the gaps below 360 px.

**Fixed when:** In Hebrew today comes first on the right, labels stay on one line, and all three days fit at 320 px.

**Check:**
```json
{
 "url": "https://www.ogo.co.il/he/city/tel-aviv",
 "viewport": "mobile",
 "wait_ms": 3500,
 "assert_bug_present": [
  {
   "js_true": "[...document.querySelectorAll('span')].some(s=>/^יום [דהו]׳$/.test(s.innerText.trim())&&getComputedStyle(s).direction==='ltr')"
  }
 ]
}
```

## OBS-34 · The calendar's category filter slides under the header when you scroll  
**Priority:** should fix  
**Viewport:** mobile  
**Pages:** https://www.ogo.co.il/ru/calendar, https://www.ogo.co.il/he/calendar, https://www.ogo.co.il/en/calendar

**What we saw:** The filter row "Все · Праздники · Каникулы · Природа · Другое" is meant to stay at the top while scrolling, but it sticks at the same height as the fixed header and ends up behind it. After a few months of scrolling only the search header is visible. Same in all three languages.

**Why it matters:** On a long year list the visitor has to scroll all the way back up to change the category.

**Steps:**
1. Open /ru/calendar on a phone (390 px).
2. Scroll down a few months.

**Suggested fix:** Set the sticky row top to the header height (for example top: var(--header-h)), or render it inside the header.

**Fixed when:** While scrolling, the filter row stays visible just below the header.

**Check:**
```json
{
 "url": "https://www.ogo.co.il/ru/calendar",
 "viewport": "mobile",
 "wait_ms": 3500,
 "assert_bug_present": [
  {
   "js_true": "(async()=>{const f=document.querySelector('div.sticky.top-0');f.scrollIntoView();let p=f.parentElement;while(p&&p!==document.documentElement){const o=getComputedStyle(p).overflowY;if((o==='auto'||o==='scroll')&&p.scrollHeight>p.clientHeight)break;p=p.parentElement}(p&&p!==document.documentElement?p:window).scrollBy(0,900);await new Promise(r=>setTimeout(r,800));const r=f.getBoundingClientRect();return !f.contains(document.elementFromPoint(r.left+r.width/2,r.top+r.height/2))})()"
  }
 ]
}
```

## OBS-35 · Holiday dates, event names and tour cities are cut off on phones  
**Priority:** should fix  
**Viewport:** mobile  
**Pages:** https://www.ogo.co.il/ru, https://www.ogo.co.il/ru/city/tel-aviv, https://www.ogo.co.il/ru/concerts, https://www.ogo.co.il/ru/holidays/hanukkah, https://www.ogo.co.il/en

**What we saw:** At 320 px: on the home page the "Смотрят сейчас" (being viewed now) banner shows "Фестиваль света «…" and loses the event name. On /ru/city/tel-aviv the holiday banner shows "Ханука · 4–12 дека…" and "через 9 дней · 298 собы…". On tour cards the list of cities is cut at about half its length, for example "Явне, Ашдод, Йехуд-Моноссон и ещё 30 городов" on the home page; 7 cards on /ru/concerts and 6 on /ru/holidays/hanukkah. At 390 px the Tel Aviv subtitle is still cut. At 768 px the home banners cut "Ханука · 4–12 декабря" and "22 выставки — например…".

**Why it matters:** The cut part is the content: the date, the event name, or the cities where a tour plays.

**Steps:**
1. Open /ru and /ru/city/tel-aviv at 320 px wide.
2. Read the yellow and pink banners and the city lines on tour cards.

**Suggested fix:** Use two-line clamping instead of single-line truncation on banner titles and subtitles and on tour city lists, or shorten the subtitle on narrow screens.

**Fixed when:** At 320 px the banners show the full date and event name, and tour cards show the city list on two lines or in a shorter form.

**Check:**
```json
{
 "url": "https://www.ogo.co.il/ru/city/tel-aviv",
 "viewport": "320x640",
 "wait_ms": 3500,
 "assert_bug_present": [
  {
   "js_true": "[...document.querySelectorAll('span.truncate')].some(e=>/Ханука|событий/.test(e.innerText)&&e.scrollWidth>e.clientWidth+1)"
  }
 ]
}
```

## OBS-36 · On small phones the "Next page" button is off the screen  
**Priority:** should fix  
**Viewport:** mobile  
**Pages:** https://www.ogo.co.il/ru/city/tel-aviv, https://www.ogo.co.il/ru/concerts, https://www.ogo.co.il/he/city/tel-aviv

**What we saw:** At 320 px the page numbers are wider than the screen. In Russian and English "Следующая страница" (next page) and the last page number are beyond the right edge; in Hebrew the next button is beyond the left edge and the last page shows only "2" of "72". City and concert pages, all three languages.

**Why it matters:** A visitor on a small phone cannot get past page 1 without guessing.

**Steps:**
1. Open /ru/city/tel-aviv at 320 px wide.
2. Scroll to the page numbers.

**Suggested fix:** Below 360 px show previous, current, …, last, next only, or reduce button size and gaps.

**Fixed when:** At 320 px previous and next are always visible, and the page list shrinks to fit.

**Check:**
```json
{
 "url": "https://www.ogo.co.il/ru/city/tel-aviv",
 "viewport": "320x640",
 "wait_ms": 3500,
 "assert_bug_present": [
  {
   "js_true": "[...document.querySelectorAll('nav')].some(n=>/\\d/.test(n.innerText)&&n.scrollWidth>n.clientWidth+20)"
  }
 ]
}
```

## OBS-37 · The cookie bar covers search suggestions and, in landscape, much of the event page  
**Priority:** minor  
**Viewport:** mobile  
**Pages:** https://www.ogo.co.il/ru/event/il/tel-aviv/2026/11/26/vasily-petrenko-conduct-or-nn-shelest-pianist-sTkfTcG36sfXYbiRqmqYS4, https://www.ogo.co.il/ru

**What we saw:** On a phone held sideways (568×320) the ticket bar and the cookie bar together take 106 of 320 px while scrolling, and 157 px at the top of the page with the header. On the home page at 320 px the cookie bar sits over the bottom of the search suggestions; sideways it hides a result row and "Все результаты…".

**Why it matters:** Until the visitor closes the cookie bar, a third of a landscape screen and part of the search results are hidden.

**Steps:**
1. Open the event page at 568×320 without closing the cookie bar.
2. Open /ru at 320×640 and type "Тель-Авив концерт" in the search box.

**Suggested fix:** Give the suggestions a max-height that stops above the cookie bar (or a higher z-index); in short landscape screens make the ticket bar one line and stack it with the cookie bar.

**Fixed when:** The search suggestions open above the cookie bar, and in landscape the ticket bar is one line and never shows together with the cookie bar.

**Check:**
```json
{
 "url": "https://www.ogo.co.il/ru/event/il/tel-aviv/2026/11/26/vasily-petrenko-conduct-or-nn-shelest-pianist-sTkfTcG36sfXYbiRqmqYS4",
 "viewport": "568x320",
 "wait_ms": 3500,
 "assert_bug_present": [
  {
   "js_true": "(()=>{let top=innerHeight;for(const e of document.querySelectorAll('body *')){const s=getComputedStyle(e);if(s.position==='fixed'||s.position==='sticky'){const r=e.getBoundingClientRect();if(r.height>0&&r.top>innerHeight/2&&r.height<innerHeight/2)top=Math.min(top,r.top)}}return innerHeight-top>innerHeight*0.3})()"
  }
 ]
}
```

## OBS-38 · The menu, language and cookie buttons are smaller than a fingertip  
**Priority:** minor  
**Viewport:** mobile  
**Pages:** https://www.ogo.co.il/ru, https://www.ogo.co.il/he, https://www.ogo.co.il/en

**What we saw:** The menu button is 32×32 px, the language button 38×32, the quick chips ("Сегодня", "Выходные", "Рядом со мной", "С детьми", "Ещё") 32 px tall, the cookie bar's close button 28×28. The usual minimum is 44×44.

**Why it matters:** Visitors on phones mis-tap, especially on the cookie close button.

**Steps:**
1. Open /ru at 390 px wide.
2. Measure the menu, language and cookie close buttons.

**Suggested fix:** Pad the hit area to 44 px (min-height and min-width, or a transparent ::after) without changing the visible size.

**Fixed when:** These controls have a tap area of at least 44×44 px (the drawing can stay the same size).

**Check:**
```json
{
 "url": "https://www.ogo.co.il/ru",
 "viewport": "mobile",
 "wait_ms": 3500,
 "assert_bug_present": [
  {
   "js_true": "document.querySelector('button[aria-label=\"Меню\"]').getBoundingClientRect().height<44"
  }
 ]
}
```

## OBS-39 · With a keyboard, the open menu lets you open the language picker behind it  
**Priority:** minor  
**Viewport:** mobile  
**Pages:** https://www.ogo.co.il/en

**What we saw:** Pressing Tab, Enter opens the menu. The next Tab moves focus to the "EN" button behind the menu, and Enter opens the language picker behind the open menu.

**Why it matters:** A keyboard or screen-reader user ends up in a dialog they cannot see.

**Steps:**
1. Open /en on a phone-sized screen with a keyboard.
2. Press Tab, Enter, Tab, Enter.

**Suggested fix:** Trap focus inside the open drawer (inert on the rest of the page) and return focus to the menu button on close.

**Fixed when:** While the menu is open, Tab moves only within the menu, and Escape closes it.

**Check:** none automatic; re-test by hand with the steps above.

## OBS-40 · A mistyped date in a link crashes the page  
**Priority:** fix first  
**Viewport:** desktop and mobile  
**Pages:** https://www.ogo.co.il/ru/filtered?start=2026-02-30&end=2026-02-30, https://www.ogo.co.il/ru/filtered?start=abc&end=abc, https://www.ogo.co.il/ru/filtered?start=2026-11-30&end=2026-11-26

**What we saw:** The server answers with error 500 and the page shows only "Application error: a client-side exception has occurred (see the browser console for more information)." Same for "start=abc" and "start=2026-13-01". A reversed range (start=2026-11-30&end=2026-11-26) is accepted and shown as "30 нояб. – 26 нояб." with 7 events.

**Why it matters:** A shared or edited link with one wrong digit gives the visitor a blank error page.

**Steps:**
1. Open /ru/filtered?start=2026-02-30&end=2026-02-30.

**Suggested fix:** Validate start and end on the server (ISO date, real calendar day); drop or redirect on failure; swap a reversed range.

**Fixed when:** An invalid date is ignored or corrected and the list loads normally; a reversed range is swapped.

**Check:**
```json
{
 "url": "https://www.ogo.co.il/ru/filtered?start=2026-02-30&end=2026-02-30",
 "viewport": "desktop",
 "wait_ms": 8000,
 "assert_bug_present": [
  {
   "text_present": "Application error"
  }
 ]
}
```

## OBS-41 · English and Hebrew event pages tell search engines the city and language in Russian  
**Priority:** should fix  
**Viewport:** desktop  
**Pages:** https://www.ogo.co.il/en/event/il/ashdod/2026/11/25/miyazakis-world-music-performed-by-symphonians-5wq4HyCADgX9L5ikmWdqts, https://www.ogo.co.il/he/event/il/tel-aviv/2026/11/25/light-festival-lumagica-sg3haMfwLL4yDeJsz22a12

**What we saw:** The visible English page is correct. The hidden page data that search engines read says "addressLocality": "Ашдод", "inLanguage": "Другой язык" (other language) and "keywords": "концерт". The Hebrew Lumagica page says "Тель-Авив" and "Иврит". On Russian pages too, the language is a Russian word, not a code like "he".

**Why it matters:** Google may show Russian city names in English and Hebrew results, or file the events under the wrong language.

**Steps:**
1. Open an /en/event/ or /he/event/ page.
2. Read script[type="application/ld+json"]: location.address.addressLocality, inLanguage, keywords.

**Suggested fix:** Pass the page locale into the JSON-LD builder for city and tag names; map the event language to ISO codes.

**Fixed when:** On English and Hebrew pages the page data gives the city in the page language and the language as a code (ru, he, en).

**Check:**
```json
{
 "url": "https://www.ogo.co.il/en/event/il/ashdod/2026/11/25/miyazakis-world-music-performed-by-symphonians-5wq4HyCADgX9L5ikmWdqts",
 "viewport": "desktop",
 "wait_ms": 2500,
 "assert_bug_present": [
  {
   "js_true": "[...document.querySelectorAll('script[type=\"application/ld+json\"]')].some(s=>/addressLocality\":\"[А-Яа-я]/.test(s.textContent))"
  }
 ]
}
```

## OBS-42 · About one in five event addresses has letters missing  
**Priority:** minor  
**Viewport:** desktop  
**Pages:** https://www.ogo.co.il/ru, https://www.ogo.co.il/ru/event/il/ein-hod/2026/12/04/quest-ur-rtists-village-postponed-bg5EQVChNhK4LGVwDrCGNq

**What we saw:** 583 of 2,836 event addresses look garbled: "…/st-up-sony-wright-tel-viv-…", "…/tour-riel-university-…", "…/shluvot-gathering-f-women-coping-…", "…/quest-ur-rtists-village-postponed-…". Short words such as "a", "to" and "or" seem to be removed inside other words, not only as whole words. The Lumagica event is also served at two addresses, dated 12 and 25 November.

**Why it matters:** A visitor who shares or reads a link sees a broken word, and the same event under two addresses splits its search ranking.

**Steps:**
1. Hover any event link on /ru and read the address.

**Suggested fix:** Remove stop-words only as whole tokens (split on "-", filter, join); add one canonical URL per event id.

**Fixed when:** Addresses keep whole words ("tel-aviv", "tour-ariel-university") and each event has one address.

**Check:**
```json
{
 "url": "https://www.ogo.co.il/ru",
 "viewport": "desktop",
 "wait_ms": 2500,
 "assert_bug_present": [
  {
   "js_true": "[...document.querySelectorAll('a[href*=\"/event/\"]')].some(a=>/tel-viv|-ur-|riel-university|-f-/.test(a.getAttribute('href')))"
  }
 ]
}
```

## OBS-43 · The brand kit offers two downloads both labelled "PNG"  
**Priority:** minor  
**Viewport:** desktop  
**Pages:** https://www.ogo.co.il/en/partners

**What we saw:** The Avatar card shows "PNG | PNG". The two links download ogo-avatar-blue-1024.png and ogo-avatar-white-1024.png.

**Why it matters:** A partner has to download both to find the one they need.

**Steps:**
1. Open /en/partners.
2. Read the Avatar card.

**Suggested fix:** Add the colour to each download label.

**Fixed when:** The links read "Blue PNG" and "White PNG".

**Check:**
```json
{
 "url": "https://www.ogo.co.il/en/partners",
 "viewport": "desktop",
 "wait_ms": 2500,
 "assert_bug_present": [
  {
   "js_true": "(async()=>{const b=[...document.querySelectorAll('button,a,[role=tab]')].find(e=>e.innerText.trim()==='Brand materials');if(b){b.click();await new Promise(r=>setTimeout(r,2000))}return /Avatar\\s*\\n\\s*PNG\\s*\\n\\s*PNG/.test(document.body.innerText)})()"
  }
 ]
}
```
