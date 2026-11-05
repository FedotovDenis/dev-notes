PLAN.md — The Movies App (контрольная Okten)

Что это: рабочий план с галочками. В конце сессии агент ставит [x] у сделанного. Файл не пушится (в .gitignore), живая копия-зеркало — dev-notes/movies-app/PLAN.md.

ЦЕЛЬ (по ТЗ)
Фильмотека на TMDB API: список фильмов + пагинация, фильтр по жанрам, поиск по названию или части, сортировка, детальная страница фильма (роутинг). Проходной 75%, 2 попытки.

ЗАФИКСИРОВАННЫЕ РЕШЕНИЯ

React 18 + TypeScript + Vite, React Router v6 (createBrowserRouter + RouterProvider)
axios-инстанс: baseURL + Bearer-интерцептор (паттерн — AGENTS.md, раздел 8)
Токен в .env: VITE_API_BASE_URL, VITE_TMDB_TOKEN (AGENTS.md, раздел 4)
Стейт: локальный useState + кастомный хук; Redux не требуется (опция «для блеска» после Этапа 8)
Всё состояние — в query-параметрах URL: ?page= ?genre= ?query= ?sort= (шарящиеся ссылки, AGENTS.md, раздел 5)
Работаем в main, без веток (по ТЗ)
Стили: обычный CSS, по файлу на компонент
Поиск: /search/movie (НЕ /search/keyword — ловушка из постмана!)
СТРУКТУРА (по ТЗ)
src/

components/ — Header, MoviesList, MoviesListCard, PosterPreview, StarsRating, MovieInfo, GenreBadge, UserInfo
containers/ — MoviesPage (основной), MovieDetailsPage
layouts/ — MainLayout (Header + Outlet)
services/ — api.service.ts
models/ — IMovie.ts, IGenre.ts
hooks/ — useMovies.ts
constants/ — urls.ts
main.tsx — роуты: / и /movie/:id
ГРАФИК (обновлён 05.11.26)

Чт 5.11 вечер: закрыть Этап 0 (PLAN на место, зеркало в dev-notes)
Пт 6.11 вечер (~1 час): Этап 1
Сб 7.11: смена 9–18, вечер — отдых
Вс 8.11 (выходной, полный день): Этапы 2–4
Пн 9.11 (выходной, полный день): Этапы 5–7
Вт 10.11 вечер: Этап 8
Ср 11.11: буфер, сдача
ЭТАП 0 — ПОДГОТОВКА
[x] Репо movies-app на GitHub (public, пустое)
[x] Vite react-ts + зависимости (TS 5.9.2 после отката 7.0.2)
[x] .gitignore со служебными файлами ДО первого коммита (проверено: в коммите be09de3 их нет)
[x] Первый пуш в main
[x] AGENTS.md и WORKFLOW.md скопированы из dev-notes
[x] Аккаунт themoviedb.org + API Read Access Token
[x] .env: VITE_API_BASE_URL + VITE_TMDB_TOKEN
[x] PLAN.md в корне проекта
[x] Зеркало dev-notes/movies-app/ + пуш dev-notes

ЭТАП 1 — КАРКАС И РОУТИНГ (на моках)
[ ] Мок: 3 фильма захардкодить (id, title, poster_path, vote_average, genre_ids)
[ ] MainLayout: Header + Outlet
[ ] Header: логотип + место под поиск + UserInfo
[ ] UserInfo: круг-аватар + имя (хардкод, по ТЗ)
[ ] Роуты: / → MoviesPage, /movie/:id → MovieDetailsPage (заглушка)
[ ] MoviesList рендерит MoviesListCard (map + key!)
[ ] MoviesListCard = PosterPreview + MovieInfo (название, обрезанное описание, бейджи)
[ ] StarsRating: vote_average (0–10) делим на 2 → 5 звёзд
[ ] GenreBadge: свой чип-бейдж (3 строки CSS)
[ ] Карточка обёрнута в Link на /movie/:id
[ ] CSS-грид: 4–5 колонок

ЭТАП 2 — ЖИВЫЕ ДАННЫЕ
[ ] Прогнать запросы глазами в Postman: discover, genres, search/movie, movie/{id}
[ ] api.service.ts: axios-инстанс + интерцептор Bearer из import.meta.env
[ ] getMovies(page, genre, sort) → /discover/movie (+with_genres, +sort_by)
[ ] getGenres() → /genre/movie/list
[ ] getMovie(id) → /movie/{id}
[ ] searchMovies(query, page) → /search/movie?query=
[ ] models: IMovie, IGenre
[ ] hooks: useMovies (загрузка + loading + error)
[ ] Постеры: https://image.tmdb.org/t/p/w500 + poster_path, заглушка при null
[ ] Жанры на карточках: genre_ids → имена через мапу

ЭТАП 3 — ПАГИНАЦИЯ
[ ] ?page= через useSearchParams
[ ] Кнопки «‹ Назад / Вперёд ›» + «страница N из total_pages»
[ ] Границы: не ниже 1, не выше total_pages
[ ] Скролл наверх при смене страницы

ЭТАП 4 — ЖАНРЫ (фильтр)
[ ] Клик по GenreBadge на карточке → ?genre=ID → discover?with_genres=ID
[ ] Индикатор активного жанра + сброс («Все» / ×)
[ ] Смена жанра сбрасывает page → 1

ЭТАП 5 — ПОИСК
[ ] Инпут в Header, Enter/кнопка → ?query=...
[ ] Режим: есть query → /search/movie; нет → /discover (+genre +sort)
[ ] Пустой результат → «ничего не найдено»
[ ] Очистка запроса → возврат к обычному списку
[ ] Пагинация работает и в поиске

ЭТАП 6 — СОРТИРОВКА
[ ] Select: популярность / рейтинг / дата выхода / название
[ ] ?sort= → sort_by: popularity.desc, vote_average.desc, primary_release_date.desc, original_title.asc
[ ] Смена сортировки сбрасывает page → 1
[ ] Бонус: при «по рейтингу» добавить vote_count.gte=300 (иначе наверху мусор)
[ ] В поиске не работает (релевантность) — норма API, не баг

ЭТАП 7 — ДЕТАЛЬНАЯ СТРАНИЦА
[ ] useParams → getMovie(id)
[ ] Раскладка: большой постер, заголовок, слоган, жанры, рейтинг, дата, длительность, полный overview
[ ] Кнопка «Назад к списку»
[ ] Ошибки: фильм не найден / сеть

ЭТАП 8 — ПОЛИРОВКА И СДАЧА
[ ] 404-роут
[ ] Спиннеры или аккуратные скелетоны
[ ] Адаптив: 1–2 колонки на мобиле
[ ] Ховеры карточек
[ ] Сверка с ТЗ пункт-в-пункт
[ ] npm run build: 0 ошибок
[ ] README для проверяющего: описание, стек, запуск (.env + токен, npm i, npm run dev), структура
[ ] git status чистый, служебных файлов нет нигде
[ ] Ссылка на репо — ментору

ОТКРЫТЫЕ ВОПРОСЫ

Точный дедлайн сдачи — уточнить у ментора (ориентир ~11.11)
Потребуют ли деплой (GitHub Pages) — уточнить там же
Redux «для блеска» — строго после Этапа 8
