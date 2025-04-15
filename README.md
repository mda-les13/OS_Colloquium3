# OS_Colloquium3

### Шаблон Состояние (State)

**Проблема:** Необходимо управлять поведением объекта в зависимости от его внутреннего состояния без использования множества условных операторов.

**Решение:** Инкапсулируют различные состояния объекта в отдельные классы и делегируют им обработку поведения объекта в зависимости от текущего состояния. Это позволяет добавлять новые состояния без изменения существующего кода.

**Примеры использования:**

1. **АТМ (Автоматическая Торговая Машина):**
   - **Проблема:** АТМ должна менять свое поведение в зависимости от состояния (например, работа, обслуживание, недостаток средств).
   - **Решение:** Каждое состояние (работа, обслуживание, недостаток средств) инкапсулируется в отдельный класс, и машина делегирует обработку действий текущему состоянию.
   - **Код:**
     ```cpp
     class ATMState {
     public:
         virtual void insertCard(ATM* atm) = 0;
         virtual void ejectCard(ATM* atm) = 0;
         virtual void insertPin(ATM* atm, int pin) = 0;
         virtual void requestCash(ATM* atm, int cash) = 0;
     };

     class HasCard : public ATMState {
     public:
         void insertCard(ATM* atm) override {
             std::cout << "You can't enter more than one card\n";
         }
         void ejectCard(ATM* atm) override {
             std::cout << "Your card is ejected\n";
             atm->setState(atm->getNoCardState());
         }
         void insertPin(ATM* atm, int pin) override {
             if (pin == 1234) {
                 std::cout << "You entered the right PIN\n";
                 atm->setState(atm->getHasPinState());
             } else {
                 std::cout << "Wrong PIN\n";
                 atm->setState(atm->getNoCardState());
             }
         }
         void requestCash(ATM* atm, int cash) override {
             std::cout << "Enter PIN first\n";
         }
     };

     class ATM {
     private:
         ATMState* noCardState;
         ATMState* hasCardState;
         ATMState* hasPinState;
         ATMState* noCashState;
         ATMState* currentState;
         int cashInMachine;
         bool correctPinEntered;

     public:
         ATM() {
             noCardState = new NoCard(this);
             hasCardState = new HasCard(this);
             hasPinState = new HasPin(this);
             noCashState = new NoCash(this);
             currentState = noCardState;
             cashInMachine = 2000;
             correctPinEntered = false;
         }

         void setState(ATMState* newState) {
             currentState = newState;
         }

         ATMState* getNoCardState() { return noCardState; }
         ATMState* getHasCardState() { return hasCardState; }
         ATMState* getHasPinState() { return hasPinState; }
         ATMState* getNoCashState() { return noCashState; }

         void insertCard() {
             currentState->insertCard(this);
         }

         void ejectCard() {
             currentState->ejectCard(this);
         }

         void insertPin(int pin) {
             currentState->insertPin(this, pin);
         }

         void requestCash(int cash) {
             currentState->requestCash(this, cash);
         }
     };
     ```

2. **QT:**
   - **Пример:** В `QStateMachine` используются состояния для управления поведением пользовательского интерфейса в зависимости от текущего состояния системы.
   - **Код:**
     ```cpp
     #include <QCoreApplication>
     #include <QStateMachine>
     #include <QState>
     #include <QFinalState>
     #include <QDebug>

     int main(int argc, char *argv[]) {
         QCoreApplication a(argc, argv);

         QStateMachine machine;

         QState *s1 = new QState();
         QState *s2 = new QState();
         QFinalState *finalState = new QFinalState();

         s1->addTransition(s2);
         s2->addTransition(finalState);

         QObject::connect(&machine, &QStateMachine::finished, &a, &QCoreApplication::quit);

         s1->assignProperty(qDebug(), "state", "State 1");
         s2->assignProperty(qDebug(), "state", "State 2");

         machine.addState(s1);
         machine.addState(s2);
         machine.addState(finalState);

         machine.setInitialState(s1);
         machine.start();

         return a.exec();
     }
     ```

3. **WPF:**
   - **Пример:** В MVVM паттерне может использоваться шаблон State для управления состоянием представления в зависимости от данных модели.
   - **Код:**
     ```xml
     <Window x:Class="StatePatternWPF.MainWindow"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             Title="MainWindow" Height="350" Width="525">
         <Grid>
             <ContentControl Content="{Binding CurrentViewState}">
                 <ContentControl.Resources>
                     <DataTemplate DataType="{x:Type local:NormalViewState}">
                         <TextBlock Text="Normal View" />
                     </DataTemplate>
                     <DataTemplate DataType="{x:Type local:EditingViewState}">
                         <TextBlock Text="Editing View" />
                     </DataTemplate>
                 </ContentControl.Resources>
             </ContentControl>
         </Grid>
     </Window>
     ```
     ```csharp
     public class ViewModel : INotifyPropertyChanged
     {
         private object _currentViewState;
         public object CurrentViewState
         {
             get { return _currentViewState; }
             set
             {
                 _currentViewState = value;
                 OnPropertyChanged(nameof(CurrentViewState));
             }
         }

         public ViewModel()
         {
             CurrentViewState = new NormalViewState();
         }

         public event PropertyChangedEventHandler PropertyChanged;
         protected void OnPropertyChanged(string propertyName)
         {
             PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
         }
     }

     public class NormalViewState { }
     public class EditingViewState { }
     ```

### Шаблон Посредник (Mediator)

**Проблема:** Несколько объектов взаимодействуют друг с другом напрямую, что усложняет управление зависимостями и создает жесткую связь между ними.

**Решение:** Создается отдельный объект-посредник, который управляет взаимодействием между объектами. Объекты общаются только с посредником, что снижает связанность системы.

**Примеры использования:**

1. **Чат-приложение:**
   - **Проблема:** Пользователи в чате должны обмениваться сообщениями без прямого взаимодействия друг с другом.
   - **Решение:** Используется объект-посредник для управления отправкой и получением сообщений.
   - **Код:**
     ```cpp
     class ChatMediator {
     public:
         virtual void sendMessage(const std::string& message, User* user) = 0;
     };

     class User {
     protected:
         ChatMediator* mediator;
         std::string name;

     public:
         User(ChatMediator* med, const std::string& n) : mediator(med), name(n) {}

         virtual void send(const std::string& message) = 0;
         virtual void receive(const std::string& message) = 0;
     };

     class ConcreteUser : public User {
     public:
         ConcreteUser(ChatMediator* med, const std::string& n) : User(med, n) {}

         void send(const std::string& message) override {
             std::cout << name << ": Sending Message: " << message << "\n";
             mediator->sendMessage(message, this);
         }

         void receive(const std::string& message) override {
             std::cout << name << ": Received Message: " << message << "\n";
         }
     };

     class ConcreteChatMediator : public ChatMediator {
     private:
         std::vector<User*> users;

     public:
         void addUser(User* user) {
             users.push_back(user);
         }

         void sendMessage(const std::string& message, User* user) override {
             for (auto u : users) {
                 if (u != user) {
                     u->receive(message);
                 }
             }
         }
     };
     ```

2. **QT:**
   - **Пример:** В `QtSignalMapper` используется для перенаправления сигналов от нескольких источников к одному слоту, упрощая управление взаимодействием между элементами интерфейса.
   - **Код:**
     ```cpp
     #include <QApplication>
     #include <QPushButton>
     #include <QSignalMapper>
     #include <QWidget>
     #include <QVBoxLayout>
     #include <QDebug>

     int main(int argc, char *argv[]) {
         QApplication app(argc, argv);

         QWidget window;
         QVBoxLayout *layout = new QVBoxLayout(&window);

         QSignalMapper *signalMapper = new QSignalMapper(&window);

         QPushButton *button1 = new QPushButton("Button 1", &window);
         QPushButton *button2 = new QPushButton("Button 2", &window);

         layout->addWidget(button1);
         layout->addWidget(button2);

         signalMapper->setMapping(button1, 1);
         signalMapper->setMapping(button2, 2);

         QObject::connect(button1, &QPushButton::clicked, signalMapper, QOverload<>::of(&QSignalMapper::map));
         QObject::connect(button2, &QPushButton::clicked, signalMapper, QOverload<>::of(&QSignalMapper::map));

         QObject::connect(signalMapper, QOverload<int>::of(&QSignalMapper::mapped), [&](int id) {
             qDebug() << "Button" << id << "clicked";
         });

         window.setLayout(layout);
         window.show();

         return app.exec();
     }
     ```

3. **WPF:**
   - **Пример:** Команды в MVVM могут использоваться как посредники между представлением и моделью, управляя реакцией на действия пользователя.
   - **Код:**
     ```xml
     <Window x:Class="MediatorPatternWPF.MainWindow"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             Title="MainWindow" Height="350" Width="525">
         <StackPanel>
             <Button Content="Button 1" Command="{Binding ButtonCommand}" CommandParameter="1" />
             <Button Content="Button 2" Command="{Binding ButtonCommand}" CommandParameter="2" />
         </StackPanel>
     </Window>
     ```
     ```csharp
     public class RelayCommand : ICommand
     {
         private readonly Action<object> _execute;
         private readonly Predicate<object> _canExecute;

         public RelayCommand(Action<object> execute, Predicate<object> canExecute = null)
         {
             _execute = execute ?? throw new ArgumentNullException(nameof(execute));
             _canExecute = canExecute;
         }

         public bool CanExecute(object parameter)
         {
             return _canExecute == null || _canExecute(parameter);
         }

         public void Execute(object parameter)
         {
             _execute(parameter);
         }

         public event EventHandler CanExecuteChanged
         {
             add { CommandManager.RequerySuggested += value; }
             remove { CommandManager.RequerySuggested -= value; }
         }
     }

     public class ViewModel : INotifyPropertyChanged
     {
         public ICommand ButtonCommand { get; }

         public ViewModel()
         {
             ButtonCommand = new RelayCommand(OnButtonClick);
         }

         private void OnButtonClick(object parameter)
         {
             int buttonId = (int)parameter;
             Debug.WriteLine($"Button {buttonId} clicked");
         }

         public event PropertyChangedEventHandler PropertyChanged;
         protected void OnPropertyChanged(string propertyName)
         {
             PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
         }
     }
     ```

### Шаблон Наблюдатель (Observer)

**Проблема:** Необходимо организовать механизм подписки, чтобы один объект мог автоматически уведомлять другие объекты о изменениях в своем состоянии.

**Решение:** Один объект (субъект) поддерживает список зависимых от него объектов (наблюдателей) и автоматически уведомляет их о любых изменениях своего состояния.

**Примеры использования:**

1. **Погодная станция:**
   - **Проблема:** Различные устройства (дисплеи, термометры) должны обновляться при изменении данных о погоде.
   - **Решение:** Используется шаблон Observer для уведомления всех устройств о новых данных.
   - **Код:**
     ```cpp
     class Observer {
     public:
         virtual void update(float temp, float humidity, float pressure) = 0;
     };

     class Subject {
     protected:
         std::vector<Observer*> observers;

     public:
         void registerObserver(Observer* o) {
             observers.push_back(o);
         }

         void removeObserver(Observer* o) {
             auto it = std::find(observers.begin(), observers.end(), o);
             if (it != observers.end()) {
                 observers.erase(it);
             }
         }

         void notifyObservers() {
             for (auto o : observers) {
                 o->update(temperature, humidity, pressure);
             }
         }

     protected:
         float temperature;
         float humidity;
         float pressure;
     };

     class WeatherData : public Subject {
     public:
         void measurementsChanged() {
             notifyObservers();
         }

         void setMeasurements(float temperature, float humidity, float pressure) {
             this->temperature = temperature;
             this->humidity = humidity;
             this->pressure = pressure;
             measurementsChanged();
         }
     };

     class CurrentConditionsDisplay : public Observer {
     public:
         void update(float temp, float humidity, float pressure) override {
             std::cout << "Current conditions: " << temp << "F degrees and " << humidity << "% humidity\n";
         }
     };
     ```

2. **QT:**
   - **Пример:** В `QObject` используется механизм сигналов и слотов, который реализует шаблон Observer, позволяя объектам реагировать на изменения других объектов.
   - **Код:**
     ```cpp
     #include <QCoreApplication>
     #include <QObject>
     #include <QDebug>

     class WeatherStation : public QObject {
         Q_OBJECT
     public:
         void setTemperature(float temp) {
             if (temperature != temp) {
                 temperature = temp;
                 emit temperatureChanged(temp);
             }
         }

     signals:
         void temperatureChanged(float temp);

     private:
         float temperature;
     };

     class TemperatureDisplay : public QObject {
         Q_OBJECT
     public slots:
         void displayTemperature(float temp) {
             qDebug() << "Current temperature:" << temp;
         }
     };

     int main(int argc, char *argv[]) {
         QCoreApplication a(argc, argv);

         WeatherStation station;
         TemperatureDisplay display;

         QObject::connect(&station, &WeatherStation::temperatureChanged, &display, &TemperatureDisplay::displayTemperature);

         station.setTemperature(25.0);
         station.setTemperature(27.5);

         return a.exec();
     }
     ```

3. **WPF:**
   - **Пример:** Привязка данных в MVVM использует шаблон Observer для автоматического обновления представления при изменении данных модели.
   - **Код:**
     ```xml
     <Window x:Class="ObserverPatternWPF.MainWindow"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             Title="MainWindow" Height="350" Width="525">
         <StackPanel>
             <TextBox Text="{Binding Temperature, UpdateSourceTrigger=PropertyChanged}" />
             <TextBlock Text="{Binding Temperature, StringFormat='Current temperature: {0}°C'}" />
         </StackPanel>
     </Window>
     ```
     ```csharp
     public class ViewModel : INotifyPropertyChanged
     {
         private double _temperature;
         public double Temperature
         {
             get { return _temperature; }
             set
             {
                 if (_temperature != value)
                 {
                     _temperature = value;
                     OnPropertyChanged(nameof(Temperature));
                 }
             }
         }

         public ViewModel()
         {
             Temperature = 25.0;
         }

         public event PropertyChangedEventHandler PropertyChanged;
         protected void OnPropertyChanged(string propertyName)
         {
             PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
         }
     }
     ```

### Дизайн ПО и Многопоточность

**Что такое дизайн ПО:**
Дизайн программного обеспечения — это процесс создания архитектуры и структуры программы, включающий выбор подходящих алгоритмов, структур данных, компонентов и их взаимодействия. Дизайн должен обеспечивать эффективное использование ресурсов, поддерживаемость и масштабируемость системы.

**Влияние многопоточности на дизайн ПО:**

1. **Синхронизация доступа к общим ресурсам:**
   - **Проблема:** При одновременном доступе нескольких потоков к общим ресурсам может возникнуть конфликт, что приводит к некорректным результатам.
   - **Решение:** Использование примитивов синхронизации, таких как мьютексы (`std::mutex`), семафоры, условия (`std::condition_variable`), блокировки (`std::lock_guard`, `std::unique_lock`).
   - **Пример:**
     ```cpp
     #include <iostream>
     #include <thread>
     #include <mutex>

     std::mutex mtx;
     int sharedResource = 0;

     void increment() {
         std::lock_guard<std::mutex> lock(mtx);
         ++sharedResource;
         std::cout << "Shared resource: " << sharedResource << "\n";
     }

     int main() {
         std::thread t1(increment);
         std::thread t2(increment);

         t1.join();
         t2.join();

         return 0;
     }
     ```

2. **Управление жизненным циклом потоков:**
   - **Проблема:** Неправильное управление потоками может привести к утечкам ресурсов и непредвиденному поведению.
   - **Решение:** Корректное создание, управление и завершение потоков, использование `std::thread`, `std::async`, `std::future`.
   - **Пример:**
     ```cpp
     #include <iostream>
     #include <thread>
     #include <future>

     void task(int id) {
         std::cout << "Task " << id << " started\n";
         // Simulate work
         std::this_thread::sleep_for(std::chrono::seconds(1));
         std::cout << "Task " << id << " finished\n";
     }

     int main() {
         std::future<void> f1 = std::async(std::launch::async, task, 1);
         std::future<void> f2 = std::async(std::launch::async, task, 2);

         f1.get();
         f2.get();

         return 0;
     }
     ```

3. **Обработка исключений:**
   - **Проблема:** Исключения в многопоточных приложениях должны быть корректно обрабатываться, чтобы избежать непредвиденного поведения.
   - **Решение:** Использование блоков `try-catch` внутри потоков, передача исключений через `std::promise` и `std::future`.
   - **Пример:**
     ```cpp
     #include <iostream>
     #include <thread>
     #include <future>
     #include <stdexcept>

     void riskyTask(std::promise<void>& promise) {
         try {
             // Simulate risky operation
             throw std::runtime_error("Something went wrong!");
             promise.set_value();
         } catch (...) {
             promise.set_exception(std::current_exception());
         }
     }

     int main() {
         std::promise<void> promise;
         std::future<void> future = promise.get_future();

         std::thread t(riskyTask, std::ref(promise));

         try {
             future.get();
             std::cout << "Task completed successfully\n";
         } catch (const std::exception& e) {
             std::cout << "Exception caught: " << e.what() << "\n";
         }

         t.join();

         return 0;
     }
     ```
