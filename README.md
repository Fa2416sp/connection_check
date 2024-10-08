# Internet Connection Checker

Этот проект предоставляет программу на Rust, которая проверяет доступность IP-адреса `8.8.8.8` (публичный DNS Google) и записывает статус (включено или выключено) вместе с временной меткой. Программа работает на Linux.

## Функции
- Периодическая проверка состояния подключения.
- Запись состояния в файл (`connection_status.log`).
- Вывод состояния в консоль для мониторинга в реальном времени.


## Требования
- Rust (Cargo)
- Операционная система Linux

## Установка
1. **Клонируйте репозиторий**:
```Shell
git clone https://github.com/yourusername/internet-connection-checker.git
cd internet-connection-checker
```

2. **Инициализация Cargo проекта**:
```Shell
cargo init
```

3. **Добавьте зависимость** `chrono`: Откройте файл `Cargo.toml` и добавьте следующую строку в раздел `[dependencies]`:
```Shell
[dependencies]
chrono = "0.4"
```


## Использование
1. **Скомпилируйте программу**:
```Shell
cargo build
```

2. **Запустите программу**:
```Shell
cargo run
```

## Пример кода
Вот основной код для проекта:
```Rust
use std::process::Command;
use std::time::{Duration, SystemTime};
use std::thread::sleep;
use std::fs::OpenOptions;
use std::io::Write;
use chrono::prelude::*; // импорт для работы с датой и временем

fn log_status(status: &str) {
    let now = SystemTime::now();
    let datetime: DateTime<Local> = DateTime::from(now);
    let log_entry = format!("{} - {}\n", status, datetime.format("%Y-%m-%d %H:%M:%S").to_string());

    println!("{}", log_entry); // вывод в консоль

    let mut file = OpenOptions::new()
        .append(true)
        .create(true)
        .open("connection_status.log")
        .unwrap();
    
    if let Err(e) = file.write_all(log_entry.as_bytes()) {
        eprintln!("Couldn't write to file: {}", e);
    }
}

fn main() {
    let mut previous_status = String::new();

    loop {
        let output = Command::new("ping")
            .arg("-c 1")
            .arg("8.8.8.8")
            .output()
            .expect("Failed to execute command");

        let status = if output.status.success() {
            "up"
        } else {
            "down"
        };

        if previous_status != status {
            log_status(status);
            previous_status = status.to_string();
        }

        sleep(Duration::new(5, 0)); // проверка каждые 5 секунд
    }
}
```
