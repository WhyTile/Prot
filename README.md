# 🎮 Arena Action — Прототип бойової механіки (Частина 1)

**Жанр та платформа:** 3D ізометричний арена-екшен для ПК  
**Фокус проєкту:** Механіка ближнього бою (3-ударне комбо, Input Buffer, Dash-cancel із кадрами невразливості i-frames та динамічна зміна модулів зброї).

🔗 **[Грати в інтерактивний веб-прототип (GitHub Pages)](./index.html)**  
🎨 **[Ескізи варіантів арени у Figma](https://www.figma.com/design/ixcG3hgPrWhBiqXqFz9IFm/nabros?node-id=0-1&p=f&t=SN0WtQbz1qesrU7N-0)**

---

## 1. Mind Map (Карта думок концепції)

```mermaid
flowchart LR
    Root(("Arena Action: Бойова механіка"))

    Root --> B1["Керування та Комбо"]
    B1 --> B1_1["3-ударна серія атак"]
    B1 --> B1_2["Input Buffer 200 мс"]
    B1 --> B1_3["Захист від спаму клавіш"]

    Root --> B2["Маневрування Dash"]
    B2 --> B2_1["Невразливість i-frames 0.2 с"]
    B2 --> B2_2["Dash-cancel переривання атаки"]
    B2 --> B2_3["Кулдаун 0.8 с"]

    Root --> B3["Модульна зброя - клавіша Q"]
    B3 --> B3_1["Швидкий клинок - точковий урон"]
    B3 --> B3_2["Важкий спис - широкий AoE-розмах"]
    B3 --> B3_3["Миттєва зміна без паузи"]

    Root --> B4["Зворотний зв'язок - Game Feel"]
    B4 --> B4_1["Hit-stop 0.1 с"]
    B4 --> B4_2["Hit-flash ворога"]
    B4 --> B4_3["Візуальний шлейф ударів"]

    Root --> B5["Тестова арена"]
    B5 --> B5_1["4 симетричні колони"]
    B5 --> B5_2["Контроль натовпу ворогів"]
```

---

## 2. Порівняльна таблиця з аналогами

* **Hades** (2020, PC / Консолі) — https://store.steampowered.com/app/1145360/Hades/
* **Dead Cells** (2018, PC / Консолі) — https://store.steampowered.com/app/588650/Dead_Cells/
* **Hyper Light Drifter** (2016, PC / Консолі) — https://store.steampowered.com/app/257850/Hyper_Light_Drifter/

| Критерій | Мій проєкт (Arena Action) | Hades | Dead Cells | Hyper Light Drifter |
| :--- | :--- | :--- | :--- | :--- |
| **1. Основна бойова механіка** | 3-ударні серії комбо з буферизацією введення (150–200 мс) | Швидкі комбо-серії ударів та спец-атак холодною зброєю | Серії ударів зброї ближнього бою з буферизацією | Рубаючі удари мечем із комбінацією дальніх пострілів |
| **2. Механіка ухилення** | Ривок із 0.2 с невразливості (i-frames) і dash-cancel атаки | Ривок із кадрами невразливості та атакою в русі | Перекат із вікном i-frames, що скасовує замах удару | Серійний швидкий ривок із фіксованою дистанцією |
| **3. Модифікація зброї** | Миттєва зміна модулів зброї під час бою (радіус / швидкість) без паузи | Вибір благословень богів між кімнатами | Зміна предметів у слотах та пасивних афіксів | Покращення характеристик зброї на базі |
| **4. Зворотний зв'язок (Feedback)** | Hit-flash ворога (0.1 с), hit-stop та контрастні контури атак | Ефекти hit-stop, спалахи, іскри та вібрація | Виразний hit-stop, спливаючий урон, оглушення | Лаконічний спалах пікселів, відкидання та віддача |
| **5. Простір для бою** | Компактна замкнена тестова арена з 4 колонами проти хвиль | Послідовні ізометричні кімнати з пастками | Процедурні 2D-біоми з платформами | Відкриті та напіввідкриті лабіринти в ізометрії |

**Висновок:** З *Dead Cells* та *Hades* адаптовано буферизацію введення та переривання анімації удару ривком (*dash-cancel*). Відмінність проєкту — динамічна заміна модулів зброї на льоту однією клавішею без зупинки бою.

---

## 3. Блок-схема обробки дій гравця

* **Зелений** — механіка; **Жовтий** — зворотний зв'язок; **Червоний** — стани відмови.

```mermaid
flowchart TD
    Start(["Початок кадру: Опитування введення"]) --> InputCheck{"Яку дію викликано?"}

    InputCheck -->|"Кнопка Атаки"| CheckCanAttack{"Персонаж може атакувати?"}
    CheckCanAttack -->|"Ні: стан Stun"| FailStun["Відмова: Дія заблокована"]
    CheckCanAttack -->|"Так"| ExecAttack["Механіка: Розрахунок хитбоксу удару"]
    
    ExecAttack --> CheckHit{"Влучання по ворогу?"}
    CheckHit -->|"Ні: промах"| FailMiss["Відмова: Повний вихід у Recovery"]
    CheckHit -->|"Так"| FeedbackHit["Зворотний зв'язок: Hit-stop 0.1с + Hit-flash + звук"]
    
    FeedbackHit --> ApplyDamage["Механіка: Зменшення здоров'я ворога"]
    ApplyDamage --> BufferWindow{"Введено атаку в буфер 200мс?"}
    BufferWindow -->|"Так"| NextCombo["Механіка: Перехід на наступний удар комбо"]
    BufferWindow -->|"Ні"| StateIdle["Механіка: Повернення до Idle"]

    InputCheck -->|"Ривок Space"| CheckDashCD{"Кулдаун ривка минув?"}
    CheckDashCD -->|"Ні: перезарядка"| FailCD["Відмова: Ривок заблоковано кулдауном"]
    CheckDashCD -->|"Так"| ExecDash["Механіка: Dash-cancel дій + i-frames 0.2с"]
    ExecDash --> FeedbackDash["Зворотний зв'язок: Шлейф руху + звук маневру"]
    FeedbackDash --> StateIdle

    InputCheck -->|"Зміна модуля Q"| ExecSwitch["Механіка: Перемикання радіуса та шкоди"]
    ExecSwitch --> FeedbackSwitch["Зворотний зв'язок: Оновлення іконки зброї в HUD"]
    FeedbackSwitch --> StateIdle

    classDef green fill:#d4edda,stroke:#28a745,stroke-width:2px,color:#155724;
    classDef yellow fill:#fff3cd,stroke:#ffc107,stroke-width:2px,color:#856404;
    classDef red fill:#f8d7da,stroke:#dc3545,stroke-width:2px,color:#721c24;

    class ExecAttack,ApplyDamage,NextCombo,ExecDash,ExecSwitch,StateIdle green;
    class FeedbackHit,FeedbackDash,FeedbackSwitch yellow;
    class FailStun,FailMiss,FailCD red;
```

---

## 4. Псевдокод ігрового циклу

```javascript
while (player.isAlive) {
    input = readPlayerInput();
    if (input.dash && dashCooldown <= 0) {
        triggerDash(duration = 0.2, iFrames = true); // ривок з невразливістю
        dashCooldown = 0.8;
    } 
    else if (input.attack && player.canAttack) {
        hitbox = weaponModule.calculateArea();
        if (hitbox.intersects(enemies)) {
            applyFeedback(HitStop(0.1), HitFlash(), SoundEffect());
            damageEnemies(weaponModule.baseDamage);
            comboBufferTimer = 0.2; // вікно для введення наступного удару
        }
    }
    if (input.switchModule) weaponModule.swapNext();
    updateCooldownsAndTimers();
}
```

---

## 5. Діаграма станів (FSM)

* **Зелений** — робочі стани механіки; **Жовтий** — стан зворотного зв'язку; **Червоний** — стани відмови.

```mermaid
flowchart LR
    Start(( )) --> Idle(["Idle: Очікування"])
    
    Idle -->|"WASD"| Move(["Move: Переміщення"])
    Move -->|"Зупинка"| Idle

    Idle -->|"ЛКМ або J"| Attack(["Attack: Комбо 1-2-3"])
    Move -->|"ЛКМ або J"| Attack
    Attack -->|"Клік у буфер 200мс"| Attack
    Attack -->|"Завершення"| Idle

    Idle -->|"Space"| Dash(["Dash: i-frames 0.2с"])
    Move -->|"Space"| Dash
    Attack -->|"Dash-cancel"| Dash
    Dash -->|"Кінець ривка"| Idle

    Attack -->|"Колізія з ворогом"| HitConfirm(["HitConfirm: Hit-stop 0.1с"])
    HitConfirm -->|"Продовження"| Attack

    Idle -->|"Отримано урон"| Stagger(["Stagger: Оглушення"])
    Move -->|"Отримано урон"| Stagger
    Attack -->|"Отримано урон"| Stagger

    Stagger -->|"Кінець стану"| Idle
    Stagger -->|"HP = 0"| Dead(["Dead: Кінець гри"])

    classDef green fill:#d4edda,stroke:#28a745,stroke-width:2px,color:#155724;
    classDef yellow fill:#fff3cd,stroke:#ffc107,stroke-width:2px,color:#856404;
    classDef red fill:#f8d7da,stroke:#dc3545,stroke-width:2px,color:#721c24;

    class Idle,Move,Attack,Dash green;
    class HitConfirm yellow;
    class Stagger,Dead red;
```

---

## 6. Ескізи 3 варіантів компонування арени (Top-down)

* **Варіант 1 (Відкрита кругла арена):** відхилено через монотонний біг по колу (кайтинг).
* **Варіант 2 (Арена з 4 колонами — ОБРАНО):** колони дозволяють розбивати групу ворогів під AoE-модуль та використовувати ривок за укриття.
* **Варіант 3 (Т-подібний коридор):** відхилено через блокування персонажа в глухих кутах.

🎨 **[Переглянути ескізи 3 варіантів арени у Figma](https://www.figma.com/design/ixcG3hgPrWhBiqXqFz9IFm/nabros?node-id=0-1&p=f&t=SN0WtQbz1qesrU7N-0)**

---

## 7. Джерела, ліцензії та академічна доброчесність

1. **Код бойової системи, State Machine та веб-прототип (`index.html`):** створено мною.
2. **Схеми, діаграми станів та ескізи арени ([Figma](https://www.figma.com/design/ixcG3hgPrWhBiqXqFz9IFm/nabros?node-id=0-1&p=f&t=SN0WtQbz1qesrU7N-0)):** створено мною (Mermaid / SVG).
3. **Шрифт інтерфейсу:** системний стек sans-serif / Roboto (Google Fonts, Apache License 2.0 — https://fonts.google.com/specimen/Roboto).
4. **Цільовий рушій:** Unity (Unity Personal License — https://unity.com).
5. **Використання генеративного ШІ (Google Gemini):** використано для структурування текстового опису, шаблонів діаграм Mermaid та налагодження обробки клавіш `KeyboardEvent.code` у веб-прототипі. Таймінги механік (i-frames 0.2 с, кулдаун 0.8 с, буфер 200 мс) та логіку модулів налаштовано вручну.
