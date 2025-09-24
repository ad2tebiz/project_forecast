### project_forecast

Проект для построения прогноза по данным о количестве пассажиров с использованием библиотек prophet, statsmodels и pmdarima, а также для записи получившихся прогнозных данных в таблицу БД

### Установка и запуск проекта 

# Перед началом убедитесь, что у вас настроена среда WSL2 с дистрибутивом Ubuntu 22.04 и установлен Visual Studio Code с расширением WSL, а также скачан Docker Desktop

# Откройте терминал Ubuntu WSL и выполните клонирование репозитория
git clone https://github.com/ad2tebiz/project_forecast.git

# Запустите VS Code и установите следующие расширения VS-Code (можно использовать перечисленные ниже команды в терминале или найти в списке расширений и установить вручную):
code --install-extension ms-python.python
code --install-extension ms-python.vscode-pylance
code --install-extension ms-azuretools.vscode-docker
code --install-extension formulahendry.code-runner
code --install-extension mtxr.sqltools
code --install-extension ms-vscode-remote.remote-wsl

# Подключитесь к WSL через 
1. ctrl+shift+p connect to WSL
2. ctrl+shift+p open folder in WSL и выбрать папку проекта
3. откроется папка проекта, можно открыть терминал и пользоваться

# Установите uv, если не установлен
uv –version                                                 проверка
curl -LsSf https://astral.sh/uv/install.sh | sh             установка uv (если не установлен)
echo 'export PATH="$HOME/.cargo/bin:$PATH"' >> ~/.bashrc	добавление uv в PATH
source ~/.bashrc							                применение изменений

# Установите postgresql, если не установлен
sudo apt install postgresql postgresql-contrib
psql --version      проверка

# Скопируйте и отредактируйте файл .env
cp .env.example .env

# Запуск docker
Откройте программу docker desktop
В терминале vs code запустите команду docker-compose up -d
При завершении используйте команду docker-compose down

# Подключитесь к БД через DBeaver 
Создайте таблицу
CREATE TABLE IF NOT EXISTS forecasts (
            id SERIAL PRIMARY KEY,
            forecast_date DATE NOT NULL,
            predicted_value DECIMAL(15,4) NOT NULL,
            confidence_lower DECIMAL(15,4),
            confidence_upper DECIMAL(15,4),
            model_name VARCHAR(100) NOT NULL,
            created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
            UNIQUE(forecast_date, model_name)
        )

# Включить отображение графиков
export DISPLAY=:0

### Использование

# Запуск скриптов
uv run src/forecast-prophet.py
uv run src/forecast-statsmodels.py
uv run src/forecast-pmdarima.py

# Возможная ошибка, связанная с несовместимостью версий numpy и pmdarima
Если при запуске скрипта с библиотекой pmdarima возникает ошибка связанная с несовместимостью версий numpy и pmdarima, то переустановите numpy и pmdarima
uv add "numpy==1.24.3" "pmdarima==2.0.3" "pandas==2.0.3"

Если и это не сработает, то можно установить Miniconda (который будет контролировать версии пакетов). Инструкция для установки Miniconda:
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh
source ~/.bashrc
conda create -n timeseries -c conda-forge python=3.9 pmdarima pandas matplotlib python-dotenv numpy=1.23 -y
conda tos accept --override-channels --channel https://repo.anaconda.com/pkgs/main
conda tos accept --override-channels --channel https://repo.anaconda.com/pkgs/r
conda activate timeseries

python src/forecast_pmdarima.py                     запуск скрипта
conda install numpy=1.23.5 --force-reinstall -y     переустановка при необходимости