# Создание Wasm приложения для браузера с помощью Rust
Rust уже поддерживает цель компиляции `wasm32-unknown-unknown`, которая представляет собой результирующий wasm файл.
Для того что бы сгенерировать обвязку, которая загрузит wasm модуль в браузер используется утилита из пакета `wasm-bindgen-cli`, которая называется `wasm-bindgen`.
Нужно самому написать `.html` файл, который вызовет код инициализации из `.js` файла, который будет сгенерирован с помощью `wasm-bindgen`.
Можно также воспользоваться утилитой `wasm-pack` из пакета `wasm-pack`, которая объединяет вызовы `cargo` и `wasm-bindgen` в одну цепочку для производства `js+wasm` пакета для целевой среды

## Последовательность шагов
1. Убедиться что установлена утилита `wasm-pack`, поставить ее при отсутствии с помощью команды `cargo install wasm-pack`
1. Сгенерировать новый проект rust. Требуется проект типа библиотека, потому что `wasm-pack` не умеет работать с `bin` проектами. Команда `cargo new --lib <proj_name>`. 
Для примера в дальнейшем будет использоваться `helloworld` в качестве `<proj_name>`
1. Настроить `Cargo.toml`

    - Добавить зависимость `wasm-bindgen`
    ```
    [dependencies]
    wasm-bindgen = "0.2"
    ```
    - Выбрать тип crate в cdylib
    ```
    [lib]
    crate-type = ["cdylib"]
    ```

1. В `src/lib.rs` добавить следующее содержание:
    ```
    use wasm_bindgen::prelude::*;


    #[wasm_bindgen]
    extern {
        fn alert(s: &str);
    }

    #[wasm_bindgen]
    pub fn greet() {
        alert("Hello, World!");
    }
    ```
1. Собрать wasm файл и обвзяку для загрузки его в браузер с помощью команды `wasm-pack build --target web`. В результате в каталоге `pkg` будут созданы `helloworld_bg.wasm`, `helloworld.js`, и ряд других файлов, например `package.json` для интегрирования в систему пакетов **NPM**
1. Создать в каталоге проекта файл `index.html` с содержимым:
    ```
    <html>
        <head>
            <title>Wasm: Hello World</title>
        </head>
        <body>
            <script type="module">
                import init, {greet} from "./pkg/helloworld.js";

                await init();
                greet();

            </script>
        </body>
    </html>
    ```
1. Для того чтобы приложение могло работать, оно должно открываться в браузере по `http` или `https` протоколам, а не по `file`. Для этого можно воспользоваться пакетом `simple-http-server`, если он отсутствует, его можно поставить командой `cargo install simple-http-server`. Далее нужно запустить его в каталоге проекта `simple-http-server -i`, ключ `-i` отдает `index.html` при обращенни к каталогу. По-умолчанию `simple-http-server` запускается на порту `8000`
В качестве альтернативы можно воспользоваться командой `python3 -m http.server`