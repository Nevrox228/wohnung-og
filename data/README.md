# Данные поиска квартир и работы — для другого ИИ (Cowork, ChatGPT, автоотклик)

Всё, что находит поиск, выгружается **каждые 30 минут** в двух местах:

| где | что |
|---|---|
| `~/Downloads/Wohnung/export/` (Мак Никиты) | полные файлы; у Никиты ещё пути к резюме/портфолио для вложений |
| https://nevrox228.github.io/wohnung-og/data/ | то же самое без личного (без путей к файлам и без точных маршрутов от дома) |

Список файлов с числом записей: `data/index.json`. На каждой странице сайта сверху есть ссылки «⬇ Данные для ИИ»: весь файл, Excel или **только то, что сейчас показано фильтрами**.

| файл | что внутри | страница |
|---|---|---|
| `wohnungen.json` / `.xlsx` | квартиры у Hochschule Offenburg (Оля + Никита) | `/` |
| `olya_jobs.json` / `.xlsx` | работа Оли: Kita, Krippe, Hort, Schule, Jugend | `/jobs.html` |
| `werk_jobs.json` / `.xlsx` | Никита: Werkstudent, практика, дипломная, Junior — KI, сайты, SEO | `/werk.html` |
| `eltern_jobs.json` / `.xlsx` | родители: Марина (мало немецкого), Алексей (сад/зелень) | `/eltern.html` |

Excel — для глаз; его правки затрутся при следующей выгрузке. Решения («откликнулись», «не интересно») ставятся кнопками на сайте и приходят в поле `status`.

## Общее во всех файлах

Корень: `generated` (время выгрузки), `what`, `page`, `count`, `items` (у Никиты — `jobs`). В список попадают только **открытые** объявления, копии одного объявления с разных сайтов склеены (ссылки на копии — `also_urls`).

`status`: `open` (без отметки) · `applied` (написали/откликнулись) · `viewing_or_interview` (просмотр/собеседование) · `rejected` · `not_interested`.
Даты: `posted` — дата публикации на сайте, `first_seen` — когда нашёл наш поиск (ГГГГ-ММ-ДДTчч:мм).

## Квартиры — `wohnungen.json`

`id`, `source` (Kleinanzeigen/immowelt/…), `url`, `title`, `ort`, `plz`, `street`, `lat`/`lon`, `zimmer`, `m2`, `kalt`, `nk`, `warm` (если указана), `warm_est` (наша оценка тёплой), `kaution`, `ab` (свободна с), `moebl`, `ebk`, `km` (до Hochschule), `verdict` (наша оценка цены), `notes`, `tags`, `img` (есть фото), `desc`, `route` (дорога до Hochschule: минуты, пересадки).

**Письмо арендодателю** — шаблон Оли, менять только «N-Zimmer-Wohnung in Ort»; абзац про Neben-/Heizkosten — только если `nk` пусто; просьба прислать фото — только если `img` пусто; срок въезда не писать. Текст шаблона — кнопка «✍️ письмо» на сайте.

## Работа Оли — `olya_jobs.json`

`title`, `firma`, `ort`, `street`, `url`, `kind` (Kita / Krippe, Hort / Schule, Jugend / Heim, Общее), `ausb` (это Ausbildung/Praktikum, а не работа), `zeit` (Voll-/Teilzeit), `befristet`, `quer` (подходит без профильного образования), `km` (от Offenburg Bahnhof), `src`, `desc`.

## Никита — `werk_jobs.json`

Корень: `profile` (имя, учёба, интересы; локально ещё `attachments` — пути к Lebenslauf, Portfolio, Projektliste), `counts`, `jobs`.

| поле | что это |
|---|---|
| `queue` | **кандидат на автоотклик**: `open`, ★ ≥ 3, без C1/Muttersprache, не Junior. Список уже отсортирован: очередь → KI → ★ → рядом |
| `stars` | 1–5, насколько подходит |
| `ai_focus` | направление KI (ML, Data Science, LLM, Computer Vision …) |
| `type` | `Werkstudent` · `Praktikum` · `Abschlussarbeit` · `Junior` |
| `track` | `KI / Automatisierung` · `Web / Shop / SEO` · `Online-Marketing` · `Другое` |
| `title`, `company`, `location`, `distance_km` (от Oberkirch), `remote` | |
| `german_level` | `normal` · `sehr gut` · `C1+`; `german_quote` — цитата |
| `url`, `apply_url` (сайт фирмы, есть у части Jobbörse), `apply_email` (если в тексте), `also_urls` | |
| `description` | текст вакансии — для сопроводительного письма |

После отправки отклика программа ставит отметку «откликнулся»: `PUT https://<база отметок>/wohnung/marks/werk_<id>.json` с телом `"написали"` (адрес базы — `SYNC_URL` в `wohnung.py`). Тогда `status = applied`, `queue = false`, второй раз не отправится.

## Родители — `eltern_jobs.json`

`person` (Марина / Алексей), `title`, `firma`, `ort`, `url`, `kind` (Уборка, Отель, Кухня, Склад, Производство, Поле / сезон; у Алексея — Garten- & Landschaftsbau и т. п.), `de` — немецкий: `не нужен` · `немного` · `?` (ничего не написано) · `нужен`; `de_txt` — цитата; `prof` / `prof_txt` — просят профессию/опыт; `ru` — есть русский; `help` — помогают с языком; `agency` — через Zeitarbeit; `zeit`, `schicht` (смены), `quer` (можно без опыта); `route` — дорога от дома к 8:00 (минуты, пересадки, время выхода; без координат); `km` (от дома), `desc`.

## Как дать это другому ИИ

1. Скачай нужный JSON (или «только показанное») со страницы — либо дай ссылку `https://nevrox228.github.io/wohnung-og/data/werk_jobs.json`.
2. Приложи этот файл (`data/README.md`) — в нём описаны все поля.
3. Для откликов Никиты: «возьми `jobs` с `queue: true`, для каждой напиши Anschreiben по `description`, мой профиль — `profile`, резюме — во вложении; покажи мне на подтверждение».
