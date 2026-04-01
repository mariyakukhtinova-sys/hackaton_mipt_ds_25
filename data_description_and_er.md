# Данные и ER-диаграмма

## Коротко

Текущее состояние папки `src`:

- `award_badges.csv`
- `lessons.csv`
- `user_access_histories.csv`
- `user_activity_histories.csv`
- `user_answers.csv`
- `user_award_badges.csv`
- `user_lessons.csv`
- `user_trainings.csv`
- `users.csv`
- `users_courses.csv`
- `wk_media_view_sessions.csv`
- `wk_users_courses_actions.csv`
- `xp_awards.csv`
- `xp_ratings.csv`

По текущим MD5-хэшам **полных дубликатов больше нет**.

Самые полезные связи сейчас:

- `users_courses.course_id` -> `lessons.course_id`
- `user_award_badges.award_badge_id` -> `award_badges` (по смыслу и диапазону значений)
- `user_lessons.users_course_id` -> `user_access_histories.users_course_id`
- `wk_users_courses_actions.users_course_id` -> `user_access_histories.users_course_id`
- `user_lessons` <-> `wk_users_courses_actions` почти полностью совпадают по парам `user_id + users_course_id`

## `Unnamed: 0`

Во всех таблицах `Unnamed: 0`:

- уникален
- без пропусков
- идёт подряд от `0` до `n-1`

Это технический индекс строки. Внутри таблицы он может играть роль локального ID записи, но как внешний ключ между таблицами не используется.

## Таблицы и колонки

### `users_courses.csv`

Роль: основная таблица `user-course`.

| Колонка | Тип | Описание |
|---|---|---|
| `Unnamed: 0` | `int64` | Технический ID строки |
| `user_id` | `int64` | Идентификатор пользователя |
| `course_id` | `int64` | Идентификатор курса |
| `state` | `object` | Статус записи: `active`, `inactive` |
| `created_at` | `object/datetime` | Когда пользователь попал на курс |
| `updated_at` | `object/datetime` | Последнее обновление записи |
| `group_template_id` | `float64` | Идентификатор шаблона/группы |
| `access_finished_at` | `object/date` | Когда заканчивается доступ к курсу |
| `wk_points` | `float64` | Набранные баллы |
| `wk_max_points` | `float64` | Максимально возможные баллы |
| `wk_max_viewable_lessons` | `float64` | Сколько уроков доступно для просмотра |
| `wk_max_task_count` | `float64` | Максимальное число задач |
| `wk_officially_started_at` | `object/date` | Официальная дата старта курса |
| `wk_course_completed_at` | `object/datetime` | Дата завершения курса |

### `lessons.csv`

Роль: справочник/структура уроков внутри курса.

| Колонка | Тип | Описание |
|---|---|---|
| `Unnamed: 0` | `int64` | Технический ID строки |
| `course_id` | `int64` | Идентификатор курса |
| `conspect_expected` | `bool` | Ожидается ли теория/конспект: `True`, `False` |
| `task_expected` | `bool` | Ожидается ли задание: `True`, `False` |
| `lesson_number` | `float64` | Порядковый номер урока |
| `wk_max_points` | `float64` | Максимум баллов за урок |
| `wk_task_count` | `float64` | Число задач в уроке |
| `wk_survival_training_expected` | `bool` | Есть ли survival training: `True`, `False` |
| `wk_scratch_playground_enabled` | `bool` | Включена ли практика: `True`, `False` |
| `wk_attendance_tracking_enabled` | `bool` | Включён ли трекинг посещаемости: `True`, `False` |
| `wk_video_duration` | `float64` | Длительность видео |
| `wk_attendance_tracking_disabled_at` | `object/datetime` | Когда отключили attendance tracking |

### `user_lessons.csv`

Роль: связь пользователя с конкретным уроком внутри конкретного `users_course_id`.

| Колонка | Тип | Описание |
|---|---|---|
| `Unnamed: 0` | `int64` | Технический ID строки |
| `user_id` | `object/int` | Идентификатор пользователя |
| `lesson_id` | `object/int` | Идентификатор урока |
| `group_id` | `object/int` | Идентификатор группы |
| `video_visited` | `bool` | Заходил ли на видео: `True`, `False` |
| `translation_visited` | `bool` | Заходил ли на перевод/текст: `True`, `False` |
| `users_course_id` | `object/int` | Идентификатор записи user-course |
| `solved` | `bool` | Решён ли урок/набор задач: `True`, `False` |
| `solved_tasks_count` | `int64` | Число решённых задач |
| `wk_points` | `float64` | Баллы за урок |
| `video_viewed` | `bool` | Просмотрено ли видео: `True`, `False` |
| `wk_solved_task_count` | `float64` | Число решённых задач в wk-логике |

Комментарий: это одна из самых полезных новых таблиц, потому что она связывает `user_id`, `lesson_id` и `users_course_id`.

### `user_access_histories.csv`

Роль: периоды доступа к курсу.

| Колонка | Тип | Описание |
|---|---|---|
| `Unnamed: 0` | `int64` | Технический ID строки |
| `users_course_id` | `int64` | Идентификатор записи `user-course` |
| `access_started_at` | `object/date` | Дата начала доступа |
| `access_expired_at` | `object/date` | Дата окончания доступа |
| `activator_class` | `object` | Механизм выдачи доступа: `PremiumAccessActivator`, `RevokeAccessActivator`, `StandardAccessActivator`, `ChangeAccessDurationActivator`, `MonthPremiumAccessActivator` |

Комментарий: теперь таблица лучше вписывается в схему, потому что `users_course_id` почти полностью покрывается в `user_lessons` и `wk_users_courses_actions`.

### `wk_users_courses_actions.csv`

Роль: журнал действий в рамках `users_course_id`.

| Колонка | Тип | Описание |
|---|---|---|
| `Unnamed: 0` | `int64` | Технический ID строки |
| `user_id` | `object/int` | Идентификатор пользователя |
| `users_course_id` | `object/int` | Идентификатор записи `user-course` |
| `sourceable_id` | `float64` | Идентификатор связанного объекта |
| `action` | `object` | Действие, например `visit_video` |
| `created_at` | `object/datetime` | Время создания события |
| `updated_at` | `object/datetime` | Время обновления события |
| `lesson_id` | `float64` | Идентификатор урока, если он известен |

Комментарий: таблица хорошо дополняет `user_lessons`; по парам `user_id + users_course_id` они почти полностью совпадают.

### `user_answers.csv`

Роль: попытки и отправки ответов пользователя.

| Колонка | Тип | Описание |
|---|---|---|
| `Unnamed: 0` | `int64` | Технический ID строки |
| `user_id` | `int64` | Идентификатор пользователя |
| `task_id` | `int64` | Идентификатор задачи |
| `attempts` | `int64` | Число попыток: `0`, `1` |
| `solved` | `object/bool` | Решена ли задача: `True`, `False` |
| `points` | `float64` | Набранные баллы |
| `max_attempts` | `int64` | Максимум попыток: `1`, `2` |
| `results` | `object` | Детальные результаты проверки |
| `skipped` | `object/bool` | Пропуск: `True`, `False` |
| `resource_type` | `object` | Тип ресурса: `Lesson`, `Training`, `Homework` |
| `submitted_at` | `object/datetime` | Время отправки |
| `wk_partial_answer` | `object` | Признак частичного ответа: `True`, `False` |
| `performance` | `float64` | Нормализованная успешность: `0.0`, `0.5`, `1.0` |
| `async_check_status` | `int64` | Статус асинхронной проверки: `0`, `2` |

### `user_trainings.csv`

Роль: прохождение тренингов и тестов.

| Колонка | Тип | Описание |
|---|---|---|
| `Unnamed: 0` | `int64` | Технический ID строки |
| `user_id` | `int64` | Идентификатор пользователя |
| `training_id` | `int64` | Идентификатор тренинга |
| `solved_tasks_count` | `int64` | Сколько задач решено |
| `earned_points` | `float64` | Набранные баллы |
| `type` | `object` | Тип тренинга: `UserTrainings::LessonTraining`, `UserTrainings::RegularTraining`, `UserTrainings::OlympiadTraining` |
| `state` | `object` | Статус: `checked`, `started` |
| `submitted_answers_count` | `int64` | Число отправленных ответов |
| `started_at` | `object/datetime` | Время старта |
| `finished_at` | `object/datetime` | Время завершения |
| `attempts` | `int64` | Число попыток; в выгрузке только `1` |
| `mark` | `float64` | Оценка: `2.0`, `3.0`, `4.0`, `5.0` |
| `mark_saved_at` | `object/datetime` | Когда сохранена оценка |

### `user_award_badges.csv`

Роль: выдача бейджа пользователю.

| Колонка | Тип | Описание |
|---|---|---|
| `Unnamed: 0` | `int64` | Технический ID строки |
| `award_badge_id` | `int64` | Тип выданного бейджа: `1`, `2`, `3`, `4`, `5`, `6` |
| `user_id` | `int64` | Идентификатор пользователя |
| `created_at` | `object/datetime` | Время выдачи |

### `award_badges.csv`

Роль: справочник бейджей.

| Колонка | Тип | Описание |
|---|---|---|
| `Unnamed: 0` | `int64` | Технический ID строки |
| `name` | `object` | Системное имя бейджа: `AwardBadges::OlympiadParticipant`, `AwardBadges::Solving` |
| `title` | `object` | Название бейджа: `Олимпиадник`, `Я решаю` |
| `level` | `int64` | Уровень бейджа: `1`, `2`, `3`, `4`, `5` |
| `quota` | `int64` | Порог получения: `1`, `5`, `25`, `50`, `100`, `500` |
| `special` | `bool` | Специальный бейдж: `True`, `False` |
| `unlocked_small_image_url` | `object` | URL картинки бейджа |

### `users.csv`

Роль: профиль пользователя.

| Колонка | Тип | Описание |
|---|---|---|
| `Unnamed: 0` | `int64` | Технический ID строки |
| `last_explainer_seen_→_course` | `float64` | Последний seen explainer/course: `1.0`-`7.0` |
| `created_at` | `object/datetime` | Дата создания пользователя |
| `updated_at` | `object/datetime` | Последнее обновление записи |
| `type` | `object` | Тип пользователя: `User::Pupil`, `User::Agent` |
| `remember_created_at` | `object/datetime` | Техническое поле remember-me |
| `sign_in_count` | `int64` | Число входов |
| `current_sign_in_at` | `object/datetime` | Последний текущий вход |
| `last_sign_in_at` | `object/datetime` | Предыдущий вход |
| `grade_id` | `int64` | Класс/уровень обучения |
| `subscribed` | `bool` | Подписан ли пользователь: `True`, `False` |
| `grade_checked` | `bool` | Проверен ли класс: `True`, `False` |
| `is_teacher` | `bool` | Признак преподавателя; в выгрузке только `False` |
| `timezone` | `object` | Часовой пояс |
| `grade_changed_at` | `object/datetime` | Когда менялся класс |
| `xp` | `int64` | Очки опыта |
| `d_wk_school_id` | `float64` | Идентификатор школы |
| `d_wk_municipal_id` | `float64` | Идентификатор муниципалитета |
| `d_wk_region_id` | `float64` | Идентификатор региона |
| `d_updated_at` | `object/datetime` | Техническое время обновления в DWH |
| `wk_gender` | `float64` | Пол: `1.0`, `2.0` |

Комментарий: прямого `user_id` по-прежнему нет, поэтому связь с остальными таблицами остаётся гипотетической.

### `user_activity_histories.csv`

Роль: действия по пользовательскому уроку.

| Колонка | Тип | Описание |
|---|---|---|
| `Unnamed: 0` | `int64` | Технический ID строки |
| `user_lesson_id` | `object` | Идентификатор пользовательского урока |
| `action` | `object` | Тип действия: `visit_translation`, `visit_video`, `show_conspect` |
| `created_at` | `object/datetime` | Время действия |

### `wk_media_view_sessions.csv`

Роль: просмотры медиа / вопросов.

| Колонка | Тип | Описание |
|---|---|---|
| `Unnamed: 0` | `int64` | Технический ID строки |
| `question_id` | `int64` | Идентификатор вопроса / медиа-единицы |
| `reviewed_at` | `object/datetime` | Время просмотра/проверки |
| `state` | `int64` | Технический статус: `1`, `2`, `3`, `4` |
| `count` | `object` | Число просмотров / срабатываний |
| `current_points` | `float64` | Текущие баллы |
| `recommended_points` | `float64` | Рекомендуемые баллы |

### `xp_awards.csv`

Роль: журнал начисления XP.

| Колонка | Тип | Описание |
|---|---|---|
| `Unnamed: 0` | `int64` | Технический ID строки |
| `xp` | `int64` | Начисленный XP |
| `created_at` | `object/datetime` | Время начисления |
| `user_id` | `int64` | Пользователь |

Комментарий: по смыслу это отдельная факт-таблица XP, а не дубликат `lessons.csv`, как было в старой выгрузке.

### `xp_ratings.csv`

Роль: рейтинг пользователя по XP.

| Колонка | Тип | Описание |
|---|---|---|
| `Unnamed: 0` | `int64` | Технический ID строки |
| `user_id` | `int64` | Пользователь |
| `xp` | `int64` | XP в рамках рейтинга |
| `type` | `object` | Тип рейтинга: например `XP::Ratings::AllTime`, `XP::Ratings::Weekly` |
| `created_at` | `object/datetime` | Время создания записи |
| `updated_at` | `object/datetime` | Время обновления записи |

## ER-диаграмма

```mermaid
erDiagram
    USERS_COURSES {
        int user_id
        int course_id
        string state
        datetime created_at
        datetime updated_at
        date access_finished_at
        float wk_points
        float wk_max_points
    }

    LESSONS {
        int course_id
        float lesson_number
        float wk_task_count
        float wk_max_points
    }

    USER_LESSONS {
        int user_id
        int lesson_id
        int users_course_id
        boolean video_visited
        boolean translation_visited
        boolean solved
    }

    USER_ACCESS_HISTORIES {
        int users_course_id
        date access_started_at
        date access_expired_at
        string activator_class
    }

    WK_USERS_COURSES_ACTIONS {
        int user_id
        int users_course_id
        string action
        datetime created_at
        int lesson_id
    }

    USER_ANSWERS {
        int user_id
        int task_id
        int attempts
        boolean solved
        datetime submitted_at
    }

    USER_TRAININGS {
        int user_id
        int training_id
        string state
        float mark
        datetime started_at
    }

    USER_AWARD_BADGES {
        int user_id
        int award_badge_id
        datetime created_at
    }

    AWARD_BADGES {
        int award_badge_id
        string title
        int level
    }

    USERS {
        datetime created_at
        datetime updated_at
        int sign_in_count
        float xp
    }

    XP_AWARDS {
        int user_id
        int xp
        datetime created_at
    }

    XP_RATINGS {
        int user_id
        int xp
        string type
        datetime updated_at
    }

    USERS_COURSES ||--o{ LESSONS : "course_id"
    USER_ACCESS_HISTORIES ||--o{ USER_LESSONS : "users_course_id"
    USER_ACCESS_HISTORIES ||--o{ WK_USERS_COURSES_ACTIONS : "users_course_id"
    USER_LESSONS }o--o{ WK_USERS_COURSES_ACTIONS : "user_id + users_course_id"
    USER_ANSWERS }o--o{ USERS_COURSES : "user_id only"
    USER_TRAININGS }o--o{ USERS_COURSES : "user_id only"
    USER_AWARD_BADGES }o--o{ USERS_COURSES : "user_id only"
    AWARD_BADGES ||--o{ USER_AWARD_BADGES : "award_badge_id"
    XP_AWARDS }o--o{ USERS_COURSES : "user_id only"
    XP_RATINGS }o--o{ USERS_COURSES : "user_id only"
```
