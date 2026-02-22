# AI Factory: полное руководство по работе (RU)

Этот файл собран по коду и документации репозитория `lee-to/ai-factory` (v2.0.0) и описывает практическую работу с инструментом: от установки до ежедневного цикла разработки.

## 1) Что это такое

`ai-factory` — CLI, который подготавливает окружение AI-агента в проекте:
- устанавливает набор skills (`/ai-factory.*`),
- настраивает структуру контекста в `.ai-factory/`,
- при необходимости настраивает MCP,
- задаёт spec-driven workflow: план -> реализация -> проверка -> коммиты.

CLI-команды в проекте:
- `ai-factory init`
- `ai-factory update`
- `ai-factory upgrade`

## 2) Требования

- Node.js `>=18`
- npm
- любой поддерживаемый AI-агент (Claude, Cursor, Codex CLI, OpenCode, Roo, Kilo, Gemini CLI, Copilot, Junie, и т.д.)

Проверка:
```bash
node -v
npm -v
```

## 3) Установка

Глобально:
```bash
npm install -g ai-factory
```

Без глобальной установки:
```bash
npx ai-factory init
```

## 4) Первый запуск в проекте

В корне вашего проекта:
```bash
ai-factory init
```

Что делает `init`:
1. Определяет стек (если есть `package.json`, `composer.json`, `requirements.txt` и т.д.).
2. Просит выбрать целевые агенты (можно несколько).
3. Просит выбрать базовые skills (по умолчанию выбираются все).
4. Для агентов с MCP предлагает включить:
   - GitHub MCP
   - Postgres MCP
   - Filesystem MCP
   - Chrome Devtools MCP
5. Устанавливает skills в каталог конкретного агента.
6. Сохраняет конфиг в `.ai-factory.json`.

## 5) Куда ставятся skills

В зависимости от агента:
- Claude: `.claude/skills/`
- Cursor: `.cursor/skills/`
- Codex CLI: `.codex/skills/`
- OpenCode: `.opencode/skills/`
- Roo: `.roo/skills/`
- Kilo: `.kilocode/skills/`
- Universal/прочие: `.agents/skills/`

## 6) MCP: что нужно донастроить

Если включили MCP при `init`, проверьте переменные окружения:
- GitHub MCP: `GITHUB_TOKEN`
- Postgres MCP: `DATABASE_URL`

Filesystem и Chrome Devtools MCP обычно не требуют env-переменных.

## 7) Главные рабочие команды в агенте

После `init` открываете агент и используете slash-команды:

1. Базовая инициализация контекста проекта:
```text
/aif
```

2. Планирование:
```text
//ai-factory.plan fast <описание>
//ai-factory.plan full <описание>
```
- `fast`: быстро, без ветки, план в `.ai-factory/PLAN.md`
- `full`: с git-веткой, план в `.ai-factory/plans/<branch>.md`

3. Уточнение плана (опционально):
```text
//ai-factory.improve
```

4. Выполнение задач:
```text
//ai-factory.implement
```

5. Верификация результата (рекомендуется):
```text
//ai-factory.verify
//ai-factory.verify --strict
```

6. Коммиты:
```text
//ai-factory.commit
```

## 8) Исправление багов и самообучение

Исправление:
```text
//ai-factory.fix <описание бага>
```

`//ai-factory.fix` сохраняет патчи знаний в `.ai-factory/patches/*.md`.
Эти патчи потом читаются при новых `//ai-factory.fix` и `//ai-factory.implement`, чтобы не повторять те же ошибки.

Периодически улучшайте сами skills:
```text
//ai-factory.evolve
```

## 9) Reflex Loop (итерационный режим)

Для сложных задач с quality gates:
```text
//ai-factory.loop new <задача>
//ai-factory.loop resume
//ai-factory.loop status
//ai-factory.loop list
//ai-factory.loop history <alias>
//ai-factory.loop clean <alias>
```

Состояние хранится в `.ai-factory/evolution/`.

## 10) Какие файлы появляются в проекте

Ключевые:
- `.ai-factory.json` — конфигурация AI Factory
- `.ai-factory/DESCRIPTION.md` — описание проекта
- `.ai-factory/ARCHITECTURE.md` — архитектурные правила
- `.ai-factory/PLAN.md` — быстрый план
- `.ai-factory/plans/*.md` — планы режима full
- `.ai-factory/patches/*.md` — патчи самообучения
- `.ai-factory/evolution/*` — данные `//ai-factory.loop`

## 11) Обновления и миграции

Обновить skills до актуальных:
```bash
ai-factory update
```

Миграция старого формата skills (v1 -> v2):
```bash
ai-factory upgrade
```

`upgrade` удаляет устаревшие skill-директории, ставит `/ai-factory.*` версии и сохраняет пользовательские кастомные skills.

## 12) Рекомендуемый «правильный» процесс работы

1. Один раз в проекте: `ai-factory init`.
2. В агенте: `/aif` для базового контекста.
3. На каждую фичу:
   - `//ai-factory.plan full <feature>` для серьёзной задачи,
   - `//ai-factory.improve` при необходимости,
   - `//ai-factory.implement`,
   - `//ai-factory.verify --strict`,
   - `//ai-factory.commit`.
4. На баги: `//ai-factory.fix ...`.
5. Периодически: `//ai-factory.evolve`.
6. После обновления пакета: `ai-factory update`.

## 13) Разработка самого репозитория `ai-factory`

Если вы меняете этот репозиторий как продукт:

```bash
npm install
npm run build
npm test
```

Локальная линковка CLI:
```bash
npm run link
```

Основные директории:
- `src/cli/` — команды CLI
- `src/core/` — инсталляция skills, конфиг, MCP, трансформеры агентов
- `skills/` — встроенные `/ai-factory.*` skills
- `mcp/templates/` — шаблоны MCP-конфигов
- `docs/` — пользовательская документация

## 14) Частые ошибки

- Запускают `ai-factory update` без предварительного `ai-factory init` -> нет `.ai-factory.json`.
- Ожидают, что `/aif` сразу реализует фичу -> нет, `/aif` готовит контекст.
- Не задают `GITHUB_TOKEN` / `DATABASE_URL` при включённом MCP.
- Для крупных задач используют `fast` вместо `full` и теряют удобный branch-based flow.

---

Если нужен, могу отдельно сделать вторую версию этого гайда в формате «короткий чеклист для команды» (1 страница) и добавить её в `docs/`.
