
This is an incredibly comprehensive C# curriculum. I'll provide explanations and code examples for each chapter's key concepts to help you master C#.

## **Chapter 1: Introduction and Environment Setup**

### Introduction to .NET Core Framework
.NET Core (now .NET 6/7/8+) is a cross-platform, open-source framework for building modern applications.

```csharp
// .NET Core supports multiple application types
// Console App, Web API, MVC, Blazor, etc.
// Cross-platform: Windows, Linux, macOS
```

### Creating First Console Application
```csharp
// Program.cs
Console.WriteLine("Hello, .NET World!");

// Modern .NET 6+ uses top-level statements
var name = Console.ReadLine();
Console.WriteLine($"Welcome, {name}!");
```

## **Chapter 2: C#.NET Basics**

### Data Types, Variables, and Literals
```csharp
// Value Types
int age = 25;
double salary = 50000.50;
bool isActive = true;
char grade = 'A';

// Reference Types
string name = "John Doe";
object obj = 42;

// Literals
int binary = 0b1010;  // Binary literal
int hex = 0xFF;       // Hexadecimal
double scientific = 1.5e3; // 1500

// Var keyword - implicitly typed
var city = "New York";  // Compiler infers string type
```

### Type Casting
```csharp
// Implicit casting (widening)
int num = 100;
long bigNum = num;  // Safe, no data loss

// Explicit casting (narrowing)
double pi = 3.14159;
int intPi = (int)pi;  // 3, data loss

// Using Convert class
string strNum = "123";
int convertedNum = Convert.ToInt32(strNum);

// Parse vs TryParse
if (int.TryParse("456", out int result))
{
    Console.WriteLine($"Parsed: {result}");
}

// Boxing and Unboxing
int value = 10;
object boxed = value;  // Boxing
int unboxed = (int)boxed;  // Unboxing
```

### Control Flow Statements
```csharp
// If-else
if (age >= 18)
{
    Console.WriteLine("Adult");
}
else if (age >= 13)
{
    Console.WriteLine("Teenager");
}
else
{
    Console.WriteLine("Child");
}

// Switch statement
string day = "Monday";
switch (day)
{
    case "Monday":
        Console.WriteLine("Start of work week");
        break;
    case "Friday":
        Console.WriteLine("TGIF!");
        break;
    default:
        Console.WriteLine("Regular day");
        break;
}

// Switch expression (C# 8+)
string message = day switch
{
    "Monday" => "Start of work week",
    "Friday" => "TGIF!",
    _ => "Regular day"
};

// While loop
int count = 0;
while (count < 5)
{
    Console.WriteLine($"Count: {count}");
    count++;
}

// For loop
for (int i = 0; i < 5; i++)
{
    Console.WriteLine($"Iteration: {i}");
}

// Do-while
do
{
    Console.WriteLine("Execute at least once");
} while (false);
```

### Functions and Methods
```csharp
public class Calculator
{
    // Call by value
    public int Add(int a, int b)
    {
        return a + b;
    }
    
    // Call by reference
    public void Swap(ref int a, ref int b)
    {
        int temp = a;
        a = b;
        b = temp;
    }
    
    // Out parameter
    public bool TryDivide(int a, int b, out int result)
    {
        result = 0;
        if (b == 0) return false;
        result = a / b;
        return true;
    }
    
    // Optional parameters
    public int Multiply(int a, int b = 1)
    {
        return a * b;
    }
}

// Usage
var calc = new Calculator();
int sum = calc.Add(5, 3);  // Call by value

int x = 10, y = 20;
calc.Swap(ref x, ref y);  // Call by reference

if (calc.TryDivide(10, 2, out int division))
{
    Console.WriteLine($"Result: {division}");
}
```

### Static vs Non-Static Members
```csharp
public class Student
{
    // Static field - shared across all instances
    public static int TotalStudents = 0;
    
    // Instance fields
    public string Name { get; set; }
    private int id;
    
    // Static constructor - called once
    static Student()
    {
        Console.WriteLine("Static constructor called");
    }
    
    // Instance constructor
    public Student(string name)
    {
        Name = name;
        TotalStudents++;
    }
    
    // Static method
    public static void DisplayTotal()
    {
        Console.WriteLine($"Total students: {TotalStudents}");
    }
    
    // Instance method
    public void DisplayInfo()
    {
        Console.WriteLine($"Student: {Name}");
    }
}

// Usage
Student.DisplayTotal();  // Static method call
var student = new Student("Alice");
student.DisplayInfo();  // Instance method call
```

### Nullable Types
```csharp
// Nullable value types
int? nullableInt = null;
double? nullableDouble = 3.14;

// Null-coalescing operator
int result = nullableInt ?? 0;  // Returns 0 if null

// Null-conditional operator
string name = null;
int? length = name?.Length;  // Returns null instead of throwing exception

// Nullable reference types (C# 8+)
#nullable enable
string? nullableString = null;
string nonNullableString = "Hello";
```

### Properties
```csharp
public class Person
{
    // Auto-implemented property
    public string Name { get; set; }
    
    // Read-only property
    public int Id { get; }
    
    // Computed property
    public string FullName => $"{FirstName} {LastName}";
    
    // Property with backing field
    private int age;
    public int Age
    {
        get { return age; }
        set
        {
            if (value >= 0 && value <= 120)
                age = value;
            else
                throw new ArgumentException("Invalid age");
        }
    }
    
    // Init-only property (C# 9+)
    public string SSN { get; init; }
    
    // Required property (C# 11+)
    public required string Email { get; set; }
}
```

## **Chapter 3: Object-Oriented Programming (OOP)**

### Classes, Objects, and Constructors
```csharp
public class BankAccount
{
    // Fields
    private decimal balance;
    private readonly string accountNumber;
    
    // Static constructor
    static BankAccount()
    {
        Console.WriteLine("Bank system initialized");
    }
    
    // Default constructor
    public BankAccount()
    {
        accountNumber = GenerateAccountNumber();
        balance = 0;
    }
    
    // Parameterized constructor
    public BankAccount(decimal initialBalance) : this()
    {
        balance = initialBalance;
    }
    
    // Copy constructor
    public BankAccount(BankAccount other)
    {
        accountNumber = other.accountNumber;
        balance = other.balance;
    }
    
    // Private constructor (Singleton pattern)
    private BankAccount(string specialCode)
    {
        // Special initialization
    }
    
    // Methods
    public void Deposit(decimal amount)
    {
        if (amount > 0)
            balance += amount;
    }
    
    public bool Withdraw(decimal amount)
    {
        if (amount > 0 && balance >= amount)
        {
            balance -= amount;
            return true;
        }
        return false;
    }
    
    private string GenerateAccountNumber()
    {
        return Guid.NewGuid().ToString().Substring(0, 8);
    }
}
```

### Inheritance and Polymorphism
```csharp
// Base class
public abstract class Shape
{
    public string Color { get; set; }
    
    // Abstract method - must be implemented by derived classes
    public abstract double CalculateArea();
    
    // Virtual method - can be overridden
    public virtual void Display()
    {
        Console.WriteLine($"This is a {Color} shape");
    }
    
    // Non-virtual method
    public double CalculatePerimeter()
    {
        return 0;
    }
}

// Derived class
public class Circle : Shape
{
    public double Radius { get; set; }
    
    public Circle(double radius)
    {
        Radius = radius;
    }
    
    // Override abstract method
    public override double CalculateArea()
    {
        return Math.PI * Radius * Radius;
    }
    
    // Override virtual method
    public override void Display()
    {
        base.Display();  // Call base class method
        Console.WriteLine($"Circle with radius: {Radius}");
    }
}

// Another derived class
public class Rectangle : Shape
{
    public double Width { get; set; }
    public double Height { get; set; }
    
    public Rectangle(double width, double height)
    {
        Width = width;
        Height = height;
    }
    
    public override double CalculateArea()
    {
        return Width * Height;
    }
}

// Method Overloading
public class Calculator
{
    public int Add(int a, int b) => a + b;
    public double Add(double a, double b) => a + b;
    public int Add(int a, int b, int c) => a + b + c;
}

// Operator Overloading
public class Vector
{
    public int X { get; set; }
    public int Y { get; set; }
    
    public static Vector operator +(Vector v1, Vector v2)
    {
        return new Vector { X = v1.X + v2.X, Y = v1.Y + v2.Y };
    }
    
    public static Vector operator -(Vector v1, Vector v2)
    {
        return new Vector { X = v1.X - v2.X, Y = v1.Y - v2.Y };
    }
}
```

### Interfaces and Abstract Classes
```csharp
// Interface
public interface IPaymentProcessor
{
    bool ProcessPayment(decimal amount);
    Task<bool> ProcessPaymentAsync(decimal amount);
}

public interface IRefundable
{
    bool Refund(decimal amount);
}

// Implementing multiple interfaces
public class CreditCardProcessor : IPaymentProcessor, IRefundable
{
    public bool ProcessPayment(decimal amount)
    {
        Console.WriteLine($"Processing credit card payment: ${amount}");
        return true;
    }
    
    public Task<bool> ProcessPaymentAsync(decimal amount)
    {
        return Task.FromResult(ProcessPayment(amount));
    }
    
    public bool Refund(decimal amount)
    {
        Console.WriteLine($"Refunding: ${amount}");
        return true;
    }
}

// Abstract class vs Interface
public abstract class PaymentGateway
{
    protected string ApiKey { get; set; }
    
    // Concrete method
    public void Initialize()
    {
        Console.WriteLine("Initializing payment gateway");
        LoadConfiguration();
    }
    
    // Abstract method
    protected abstract void LoadConfiguration();
    
    // Virtual method with default implementation
    public virtual bool ValidatePayment()
    {
        return true;
    }
}

// Sealed class - cannot be inherited
public sealed class SecurityManager
{
    public string Encrypt(string data)
    {
        return Convert.ToBase64String(
            System.Text.Encoding.UTF8.GetBytes(data));
    }
}

// Sealed method
public class BaseCalculator
{
    protected virtual double GetFactor()
    {
        return 1.0;
    }
}

public class AdvancedCalculator : BaseCalculator
{
    // Sealed override - prevents further overriding
    protected sealed override double GetFactor()
    {
        return 2.0;
    }
}
```

## **Chapter 5: Exception Handling**

```csharp
public class ExceptionHandlingDemo
{
    // Custom Exception
    public class BusinessRuleException : Exception
    {
        public string ErrorCode { get; }
        
        public BusinessRuleException(string message, string errorCode) 
            : base(message)
        {
            ErrorCode = errorCode;
        }
        
        public BusinessRuleException(string message, string errorCode, Exception inner) 
            : base(message, inner)
        {
            ErrorCode = errorCode;
        }
    }
    
    public void DemonstrateExceptionHandling()
    {
        // Basic try-catch
        try
        {
            int result = Divide(10, 0);
        }
        catch (DivideByZeroException ex)
        {
            Console.WriteLine($"Division error: {ex.Message}");
        }
        
        // Multiple catch blocks
        try
        {
            ProcessData();
        }
        catch (ArgumentNullException ex)
        {
            Console.WriteLine($"Null argument: {ex.Message}");
        }
        catch (BusinessRuleException ex)
        {
            Console.WriteLine($"Business rule violation: {ex.ErrorCode} - {ex.Message}");
        }
        catch (Exception ex)
        {
            Console.WriteLine($"General error: {ex.Message}");
            throw;  // Re-throw preserving stack trace
        }
        finally
        {
            Console.WriteLine("Cleanup code always executes");
        }
        
        // Exception filters
        try
        {
            ProcessTransaction(5000);
        }
        catch (BusinessRuleException ex) when (ex.ErrorCode == "INSUFFICIENT_FUNDS")
        {
            Console.WriteLine("Please add funds to your account");
        }
        catch (BusinessRuleException ex) when (ex.ErrorCode == "FRAUD_DETECTED")
        {
            Console.WriteLine("Transaction blocked for security reasons");
        }
    }
    
    // Using statement for automatic disposal
    public void ReadFile(string path)
    {
        try
        {
            using (var reader = new StreamReader(path))
            {
                string content = reader.ReadToEnd();
            }  // Reader automatically disposed
        }
        catch (FileNotFoundException)
        {
            Console.WriteLine("File not found");
        }
    }
    
    private int Divide(int a, int b)
    {
        if (b == 0)
            throw new DivideByZeroException("Cannot divide by zero");
        return a / b;
    }
    
    private void ProcessData()
    {
        throw new NotImplementedException();
    }
    
    private void ProcessTransaction(decimal amount)
    {
        if (amount > 1000)
            throw new BusinessRuleException(
                "Transaction amount exceeds limit", 
                "AMOUNT_EXCEEDED");
    }
}
```

## **Chapter 6: Delegates, Events, and Lambda Expressions**

```csharp
// Delegate declaration
public delegate int MathOperation(int a, int b);

// Generic delegates
public delegate T Operation<T>(T a, T b);

public class DelegateDemo
{
    public void DemonstrateDelegates()
    {
        // Basic delegate
        MathOperation add = (a, b) => a + b;
        MathOperation multiply = (a, b) => a * b;
        
        Console.WriteLine($"Add: {add(5, 3)}");
        Console.WriteLine($"Multiply: {multiply(5, 3)}");
        
        // Multicast delegate
        MathOperation combined = add + multiply;
        int result = combined(5, 3);  // Returns last method's result (15)
        
        // Built-in delegates
        Func<int, int, int> funcAdd = (a, b) => a + b;
        Action<string> log = message => Console.WriteLine(message);
        Predicate<int> isPositive = value => value > 0;
        
        // Anonymous method (old style)
        MathOperation subtract = delegate(int a, int b)
        {
            return a - b;
        };
        
        // Lambda expressions
        MathOperation divide = (a, b) =>
        {
            if (b == 0) throw new DivideByZeroException();
            return a / b;
        };
    }
}

// Events
public class OrderProcessor
{
    // Event declaration
    public event EventHandler<OrderEventArgs> OrderProcessed;
    public event EventHandler<OrderEventArgs> OrderFailed;
    
    // Using EventHandler<T>
    public class OrderEventArgs : EventArgs
    {
        public string OrderId { get; set; }
        public decimal Amount { get; set; }
        public DateTime ProcessedDate { get; set; }
    }
    
    public void ProcessOrder(string orderId, decimal amount)
    {
        try
        {
            Console.WriteLine($"Processing order {orderId} for ${amount}");
            
            // Business logic here
            
            // Raise event
            OnOrderProcessed(new OrderEventArgs
            {
                OrderId = orderId,
                Amount = amount,
                ProcessedDate = DateTime.Now
            });
        }
        catch (Exception ex)
        {
            OnOrderFailed(new OrderEventArgs
            {
                OrderId = orderId,
                Amount = amount
            });
        }
    }
    
    protected virtual void OnOrderProcessed(OrderEventArgs e)
    {
        OrderProcessed?.Invoke(this, e);
    }
    
    protected virtual void OnOrderFailed(OrderEventArgs e)
    {
        OrderFailed?.Invoke(this, e);
    }
}

// Subscriber
public class EmailService
{
    public void Subscribe(OrderProcessor processor)
    {
        processor.OrderProcessed += OnOrderProcessed;
        processor.OrderFailed += OnOrderFailed;
    }
    
    public void Unsubscribe(OrderProcessor processor)
    {
        processor.OrderProcessed -= OnOrderProcessed;
        processor.OrderFailed -= OnOrderFailed;
    }
    
    private void OnOrderProcessed(object sender, OrderProcessor.OrderEventArgs e)
    {
        Console.WriteLine($"Email: Order {e.OrderId} processed successfully");
    }
    
    private void OnOrderFailed(object sender, OrderProcessor.OrderEventArgs e)
    {
        Console.WriteLine($"Email: Order {e.OrderId} failed");
    }
}
```

## **Chapter 7: Multithreading**

```csharp
public class MultithreadingDemo
{
    public void DemonstrateThreading()
    {
        // Basic thread creation
        Thread thread1 = new Thread(() =>
        {
            for (int i = 0; i < 5; i++)
            {
                Console.WriteLine($"Thread 1: {i}");
                Thread.Sleep(100);
            }
        });
        
        Thread thread2 = new Thread(DoWork);
        thread2.Name = "Worker Thread";
        thread2.Priority = ThreadPriority.AboveNormal;
        
        thread1.Start();
        thread2.Start();
        
        // Wait for threads to complete
        thread1.Join();
        thread2.Join();
    }
    
    private void DoWork()
    {
        Console.WriteLine($"Thread {Thread.CurrentThread.Name} is working");
    }
    
    // Thread synchronization with lock
    private readonly object lockObject = new object();
    private int sharedCounter = 0;
    
    public void IncrementCounter()
    {
        lock (lockObject)
        {
            sharedCounter++;
        }
    }
    
    // Using Monitor for more control
    public void MonitorExample()
    {
        if (Monitor.TryEnter(lockObject, TimeSpan.FromSeconds(1)))
        {
            try
            {
                // Critical section
                sharedCounter++;
            }
            finally
            {
                Monitor.Exit(lockObject);
            }
        }
    }
    
    // Thread Pool
    public void ThreadPoolExample()
    {
        ThreadPool.QueueUserWorkItem(state =>
        {
            Console.WriteLine($"Thread pool worker: {Thread.CurrentThread.ManagedThreadId}");
        });
        
        // Get available threads
        ThreadPool.GetAvailableThreads(out int workerThreads, out int completionPortThreads);
        Console.WriteLine($"Available worker threads: {workerThreads}");
    }
    
    // Mutex for cross-process synchronization
    public async Task MutexExample()
    {
        using (var mutex = new Mutex(false, "Global\\MyAppMutex"))
        {
            if (mutex.WaitOne(TimeSpan.FromSeconds(5)))
            {
                try
                {
                    Console.WriteLine("Mutex acquired");
                    await Task.Delay(1000);
                }
                finally
                {
                    mutex.ReleaseMutex();
                    Console.WriteLine("Mutex released");
                }
            }
        }
    }
    
    // Semaphore for limiting concurrent access
    private readonly SemaphoreSlim semaphore = new SemaphoreSlim(3); // Max 3 concurrent
    
    public async Task SemaphoreExample()
    {
        await semaphore.WaitAsync();
        try
        {
            Console.WriteLine("Semaphore acquired");
            await Task.Delay(1000);
        }
        finally
        {
            semaphore.Release();
        }
    }
    
    // Deadlock prevention
    private readonly object resourceA = new object();
    private readonly object resourceB = new object();
    
    public void PreventDeadlock()
    {
        // Always acquire locks in same order
        lock (resourceA)
        {
            lock (resourceB)
            {
                // Work with both resources
            }
        }
    }
}

// Background vs Foreground threads
public class ThreadExample
{
    public void RunBackgroundThread()
    {
        Thread background = new Thread(() =>
        {
            Thread.Sleep(5000);
            Console.WriteLine("This might not execute");
        })
        {
            IsBackground = true  // Will terminate when main thread exits
        };
        background.Start();
    }
}
```

## **Chapter 8: Collections and Generics**

```csharp
public class CollectionsDemo
{
    public void DemonstrateCollections()
    {
        // Arrays
        int[] numbers = new int[] { 1, 2, 3, 4, 5 };
        int[,] matrix = new int[2, 3] { { 1, 2, 3 }, { 4, 5, 6 } };
        int[][] jagged = new int[2][];
        jagged[0] = new int[] { 1, 2 };
        jagged[1] = new int[] { 3, 4, 5 };
        
        // Generic List
        List<string> names = new List<string> { "Alice", "Bob", "Charlie" };
        names.Add("David");
        names.Remove("Bob");
        names.Sort();
        
        // Dictionary
        Dictionary<string, int> ages = new Dictionary<string, int>
        {
            ["Alice"] = 25,
            ["Bob"] = 30
        };
        
        if (ages.TryGetValue("Alice", out int age))
        {
            Console.WriteLine($"Alice is {age}");
        }
        
        // HashSet - unique elements
        HashSet<int> uniqueNumbers = new HashSet<int> { 1, 2, 3, 3, 4 };
        Console.WriteLine($"Unique count: {uniqueNumbers.Count}"); // 4
        
        // Queue and Stack
        Queue<string> queue = new Queue<string>();
        queue.Enqueue("First");
        queue.Enqueue("Second");
        string first = queue.Dequeue();
        
        Stack<int> stack = new Stack<int>();
        stack.Push(1);
        stack.Push(2);
        int top = stack.Pop();
        
        // SortedList
        SortedList<int, string> sortedList = new SortedList<int, string>
        {
            { 3, "Third" },
            { 1, "First" },
            { 2, "Second" }
        };
        
        // Concurrent collections
        ConcurrentDictionary<string, int> concurrentDict = new ConcurrentDictionary<string, int>();
        concurrentDict.TryAdd("Key", 1);
        concurrentDict.AddOrUpdate("Key", 1, (key, old) => old + 1);
        
        BlockingCollection<int> blockingCollection = new BlockingCollection<int>(boundedCapacity: 10);
        
        // Immutable collections
        ImmutableList<int> immutableList = ImmutableList.Create(1, 2, 3);
        ImmutableList<int> newList = immutableList.Add(4); // Creates new list
    }
}

// Generic constraints
public class GenericRepository<T> where T : class, IEntity, new()
{
    private List<T> items = new List<T>();
    
    public void Add(T item)
    {
        items.Add(item);
    }
    
    public T FindById(int id)
    {
        return items.FirstOrDefault(i => i.Id == id);
    }
    
    public List<T> GetAll()
    {
        return items;
    }
}

public interface IEntity
{
    int Id { get; set; }
}

// Sorting complex types
public class Person : IComparable<Person>
{
    public string Name { get; set; }
    public int Age { get; set; }
    
    public int CompareTo(Person other)
    {
        return Age.CompareTo(other.Age);
    }
}

public class PersonComparer : IComparer<Person>
{
    public int Compare(Person x, Person y)
    {
        return string.Compare(x.Name, y.Name);
    }
}
```

## **Chapter 9: File Handling**

```csharp
public class FileHandlingDemo
{
    public async Task DemonstrateFileHandling()
    {
        string filePath = @"C:\temp\example.txt";
        
        // Writing to file
        File.WriteAllText(filePath, "Hello, World!");
        File.WriteAllLines(filePath, new[] { "Line 1", "Line 2", "Line 3" });
        
        // Appending text
        File.AppendAllText(filePath, "\nAdditional text");
        
        // Reading from file
        string content = File.ReadAllText(filePath);
        string[] lines = File.ReadAllLines(filePath);
        
        // Using StreamWriter
        using (var writer = new StreamWriter(filePath, append: true))
        {
            await writer.WriteLineAsync("Async writing");
            await writer.FlushAsync();
        }
        
        // Using StreamReader
        using (var reader = new StreamReader(filePath))
        {
            string line;
            while ((line = await reader.ReadLineAsync()) != null)
            {
                Console.WriteLine(line);
            }
        }
        
        // FileStream for binary operations
        using (var fileStream = new FileStream(@"C:\temp\data.bin", FileMode.Create))
        {
            byte[] data = { 0x1, 0x2, 0x3, 0x4 };
            await fileStream.WriteAsync(data, 0, data.Length);
        }
        
        // BinaryWriter and BinaryReader
        using (var writer = new BinaryWriter(File.Open(@"C:\temp\binary.dat", FileMode.Create)))
        {
            writer.Write(42);
            writer.Write("Hello");
            writer.Write(3.14);
        }
        
        using (var reader = new BinaryReader(File.Open(@"C:\temp\binary.dat", FileMode.Open)))
        {
            int num = reader.ReadInt32();
            string str = reader.ReadString();
            double pi = reader.ReadDouble();
        }
        
        // File and Directory operations
        if (File.Exists(filePath))
        {
            FileInfo fileInfo = new FileInfo(filePath);
            Console.WriteLine($"Size: {fileInfo.Length}");
            Console.WriteLine($"Created: {fileInfo.CreationTime}");
            Console.WriteLine($"Modified: {fileInfo.LastWriteTime}");
        }
        
        if (!Directory.Exists(@"C:\temp"))
        {
            Directory.CreateDirectory(@"C:\temp");
        }
        
        string[] files = Directory.GetFiles(@"C:\temp", "*.txt");
        string[] directories = Directory.GetDirectories(@"C:\temp");
        
        // Working with JSON
        var person = new { Name = "John", Age = 30 };
        string jsonString = JsonSerializer.Serialize(person);
        
        var deserialized = JsonSerializer.Deserialize<Person>(
            jsonString,
            new JsonSerializerOptions { PropertyNameCaseInsensitive = true }
        );
        
        // Working with XML
        XElement xml = new XElement("People",
            new XElement("Person",
                new XAttribute("Id", 1),
                new XElement("Name", "John"),
                new XElement("Age", 30)
            )
        );
        xml.Save(@"C:\temp\people.xml");
        
        // Excel export (using EPPlus or ClosedXML)
        // Would need NuGet package
    }
    
    // Handling large files efficiently
    public async Task ProcessLargeFile(string inputPath, string outputPath)
    {
        const int bufferSize = 81920; // 80KB buffer
        
        using (var reader = new StreamReader(inputPath, bufferSize: bufferSize))
        using (var writer = new StreamWriter(outputPath, bufferSize: bufferSize))
        {
            char[] buffer = new char[bufferSize];
            int charsRead;
            
            while ((charsRead = await reader.ReadAsync(buffer, 0, buffer.Length)) > 0)
            {
                // Process buffer
                await writer.WriteAsync(buffer, 0, charsRead);
            }
        }
    }
}
```

## **Chapter 10: Asynchronous Programming**

```csharp
public class AsyncProgrammingDemo
{
    // Basic async/await
    public async Task<string> FetchDataAsync()
    {
        using (var client = new HttpClient())
        {
            string result = await client.GetStringAsync("https://api.example.com/data");
            return result;
        }
    }
    
    // Task-based operations
    public async Task ProcessMultipleAsync()
    {
        Task<string> task1 = FetchDataAsync();
        Task<string> task2 = FetchDataAsync();
        
        // Wait for all tasks
        string[] results = await Task.WhenAll(task1, task2);
        
        // Or wait for first to complete
        string firstResult = await Task.WhenAny(task1, task2).Result;
    }
    
    // Task cancellation
    public async Task CancellableOperationAsync(CancellationToken cancellationToken)
    {
        for (int i = 0; i < 100; i++)
        {
            cancellationToken.ThrowIfCancellationRequested();
            
            await Task.Delay(100, cancellationToken);
            Console.WriteLine($"Iteration {i}");
        }
    }
    
    public async Task UseCancellation()
    {
        using (var cts = new CancellationTokenSource(TimeSpan.FromSeconds(5)))
        {
            try
            {
                await CancellableOperationAsync(cts.Token);
            }
            catch (OperationCanceledException)
            {
                Console.WriteLine("Operation was cancelled");
            }
        }
    }
    
    // Async streams (C# 8+)
    public async IAsyncEnumerable<int> GenerateNumbersAsync(
        [EnumeratorCancellation] CancellationToken cancellationToken = default)
    {
        for (int i = 0; i < 10; i++)
        {
            await Task.Delay(100, cancellationToken);
            yield return i;
        }
    }
    
    public async Task ConsumeAsyncStream()
    {
        await foreach (var number in GenerateNumbersAsync())
        {
            Console.WriteLine(number);
        }
    }
    
    // ValueTask for high-performance scenarios
    public ValueTask<int> GetCachedValueAsync()
    {
        if (cachedValue.HasValue)
        {
            return new ValueTask<int>(cachedValue.Value);
        }
        
        return new ValueTask<int>(LoadValueAsync());
    }
    
    private int? cachedValue;
    private async Task<int> LoadValueAsync()
    {
        await Task.Delay(100);
        int value = 42;
        cachedValue = value;
        return value;
    }
    
    // Exception handling in async
    public async Task HandleAsyncExceptions()
    {
        try
        {
            await Task.WhenAll(
                Task.Run(() => throw new InvalidOperationException("Task 1 failed")),
                Task.Run(() => throw new ArgumentException("Task 2 failed"))
            );
        }
        catch (AggregateException ae)
        {
            foreach (var ex in ae.InnerExceptions)
            {
                Console.WriteLine($"Error: {ex.Message}");
            }
        }
        catch (Exception ex) when (ex is not AggregateException)
        {
            Console.WriteLine($"Single error: {ex.Message}");
        }
    }
    
    // Task chaining
    public async Task ChainTasksAsync()
    {
        string result = await Task.Run(() => "Hello")
            .ContinueWith(t => t.Result + " World")
            .ContinueWith(t => t.Result + "!");
        
        Console.WriteLine(result); // "Hello World!"
    }
}
```

## **Chapter 11: Parallel Programming**

```csharp
public class ParallelProgrammingDemo
{
    public void DemonstrateParallelism()
    {
        // Parallel.For
        Parallel.For(0, 100, i =>
        {
            Console.WriteLine($"Processing item {i} on thread {Thread.CurrentThread.ManagedThreadId}");
        });
        
        // Parallel.ForEach
        var items = Enumerable.Range(0, 100).ToList();
        Parallel.ForEach(items, item =>
        {
            ProcessItem(item);
        });
        
        // Parallel.Invoke
        Parallel.Invoke(
            () => DoTask1(),
            () => DoTask2(),
            () => DoTask3()
        );
        
        // With degree of parallelism
        var parallelOptions = new ParallelOptions
        {
            MaxDegreeOfParallelism = Environment.ProcessorCount
        };
        
        Parallel.ForEach(items, parallelOptions, item =>
        {
            ProcessItem(item);
        });
        
        // PLINQ
        var results = items.AsParallel()
            .WithDegreeOfParallelism(4)
            .Where(x => x % 2 == 0)
            .Select(x => x * 2)
            .ToList();
        
        // Ordered PLINQ
        var orderedResults = items.AsParallel()
            .AsOrdered()
            .Where(x => x % 2 == 0)
            .ToList();
    }
    
    private void ProcessItem(int item)
    {
        Thread.Sleep(10); // Simulate work
    }
    
    private void DoTask1() => Console.WriteLine("Task 1");
    private void DoTask2() => Console.WriteLine("Task 2");
    private void DoTask3() => Console.WriteLine("Task 3");
    
    // Thread-safe operations with Interlocked
    private int counter = 0;
    
    public void IncrementSafely()
    {
        Interlocked.Increment(ref counter);
        int currentValue = Interlocked.CompareExchange(ref counter, 0, 100);
        int oldValue = Interlocked.Exchange(ref counter, 50);
    }
    
    // Cooperative cancellation
    public void ParallelWithCancellation()
    {
        using (var cts = new CancellationTokenSource())
        {
            var options = new ParallelOptions
            {
                CancellationToken = cts.Token
            };
            
            // Cancel after 5 seconds
            cts.CancelAfter(5000);
            
            try
            {
                Parallel.For(0, 1000000, options, i =>
                {
                    options.CancellationToken.ThrowIfCancellationRequested();
                    ProcessItem(i);
                });
            }
            catch (OperationCanceledException)
            {
                Console.WriteLine("Operation was cancelled");
            }
        }
    }
}
```

## **Chapter 15: ADO.NET Core**

```csharp
public class ADONetDemo
{
    private readonly string connectionString = 
        "Server=localhost;Database=MyDatabase;Trusted_Connection=true;TrustServerCertificate=true";
    
    // Basic CRUD operations
    public void DemonstrateBasicOperations()
    {
        using (var connection = new SqlConnection(connectionString))
        {
            connection.Open();
            
            // Insert
            string insertSql = "INSERT INTO Users (Name, Email) VALUES (@Name, @Email)";
            using (var command = new SqlCommand(insertSql, connection))
            {
                command.Parameters.AddWithValue("@Name", "John Doe");
                command.Parameters.AddWithValue("@Email", "john@example.com");
                int rowsAffected = command.ExecuteNonQuery();
            }
            
            // Select
            string selectSql = "SELECT Id, Name, Email FROM Users WHERE Id > @Id";
            using (var command = new SqlCommand(selectSql, connection))
            {
                command.Parameters.AddWithValue("@Id", 0);
                
                using (var reader = command.ExecuteReader())
                {
                    while (reader.Read())
                    {
                        int id = reader.GetInt32(0);
                        string name = reader.GetString(1);
                        string email = reader.GetString(2);
                        Console.WriteLine($"User: {id}, {name}, {email}");
                    }
                }
            }
            
            // Update with transaction
            using (var transaction = connection.BeginTransaction())
            {
                try
                {
                    string updateSql = "UPDATE Users SET Email = @Email WHERE Id = @Id";
                    using (var command = new SqlCommand(updateSql, connection, transaction))
                    {
                        command.Parameters.AddWithValue("@Id", 1);
                        command.Parameters.AddWithValue("@Email", "updated@example.com");
                        command.ExecuteNonQuery();
                    }
                    
                    transaction.Commit();
                }
                catch
                {
                    transaction.Rollback();
                    throw;
                }
            }
        }
    }
    
    // Using stored procedures
    public void ExecuteStoredProcedure()
    {
        using (var connection = new SqlConnection(connectionString))
        {
            connection.Open();
            
            using (var command = new SqlCommand("GetUserById", connection))
            {
                command.CommandType = CommandType.StoredProcedure;
                command.Parameters.AddWithValue("@UserId", 1);
                
                // Output parameter
                command.Parameters.Add("@UserCount", SqlDbType.Int).Direction = 
                    ParameterDirection.Output;
                
                using (var reader = command.ExecuteReader())
                {
                    while (reader.Read())
                    {
                        // Process results
                    }
                }
                
                int userCount = (int)command.Parameters["@UserCount"].Value;
            }
        }
    }
    
    // DataSet and DataAdapter
    public void UseDataSet()
    {
        using (var connection = new SqlConnection(connectionString))
        {
            string selectSql = "SELECT * FROM Users";
            using (var adapter = new SqlDataAdapter(selectSql, connection))
            {
                DataSet dataSet = new DataSet();
                adapter.Fill(dataSet, "Users");
                
                DataTable usersTable = dataSet.Tables["Users"];
                foreach (DataRow row in usersTable.Rows)
                {
                    Console.WriteLine($"User: {row["Name"]}");
                }
            }
        }
    }
    
    // Bulk operations
    public async Task BulkInsertAsync(List<User> users)
    {
        using (var connection = new SqlConnection(connectionString))
        {
            await connection.OpenAsync();
            
            using (var bulkCopy = new SqlBulkCopy(connection))
            {
                bulkCopy.DestinationTableName = "Users";
                bulkCopy.BatchSize = 1000;
                bulkCopy.NotifyAfter = 500;
                
                bulkCopy.SqlRowsCopied += (sender, args) =>
                {
                    Console.WriteLine($"{args.RowsCopied} rows copied");
                };
                
                // Map columns
                bulkCopy.ColumnMappings.Add("Name", "Name");
                bulkCopy.ColumnMappings.Add("Email", "Email");
                
                DataTable table = ConvertToDataTable(users);
                await bulkCopy.WriteToServerAsync(table);
            }
        }
    }
    
    private DataTable ConvertToDataTable(List<User> users)
    {
        DataTable table = new DataTable();
        table.Columns.Add("Name", typeof(string));
        table.Columns.Add("Email", typeof(string));
        
        foreach (var user in users)
        {
            table.Rows.Add(user.Name, user.Email);
        }
        
        return table;
    }
    
    // Parameterized queries to prevent SQL injection
    public User GetUserById(int id)
    {
        using (var connection = new SqlConnection(connectionString))
        {
            connection.Open();
            
            // NEVER do this: string sql = $"SELECT * FROM Users WHERE Id = {id}";
            string sql = "SELECT Id, Name, Email FROM Users WHERE Id = @Id";
            
            using (var command = new SqlCommand(sql, connection))
            {
                command.Parameters.Add("@Id", SqlDbType.Int).Value = id;
                
                using (var reader = command.ExecuteReader())
                {
                    if (reader.Read())
                    {
                        return new User
                        {
                            Id = reader.GetInt32(0),
                            Name = reader.GetString(1),
                            Email = reader.GetString(2)
                        };
                    }
                }
            }
        }
        return null;
    }
    
    // Async operations
    public async Task<List<User>> GetAllUsersAsync()
    {
        var users = new List<User>();
        
        using (var connection = new SqlConnection(connectionString))
        {
            await connection.OpenAsync();
            
            string sql = "SELECT Id, Name, Email FROM Users";
            using (var command = new SqlCommand(sql, connection))
            using (var reader = await command.ExecuteReaderAsync())
            {
                while (await reader.ReadAsync())
                {
                    users.Add(new User
                    {
                        Id = reader.GetInt32(0),
                        Name = reader.GetString(1),
                        Email = reader.GetString(2)
                    });
                }
            }
        }
        
        return users;
    }
}

public class User
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
}
```

## **Chapter 19: LINQ**

```csharp
public class LinqDemo
{
    private List<Customer> customers;
    private List<Order> orders;
    
    public LinqDemo()
    {
        // Sample data
        customers = new List<Customer>
        {
            new Customer { Id = 1, Name = "Alice", City = "New York" },
            new Customer { Id = 2, Name = "Bob", City = "London" },
            new Customer { Id = 3, Name = "Charlie", City = "New York" }
        };
        
        orders = new List<Order>
        {
            new Order { Id = 1, CustomerId = 1, Amount = 100, Date = DateTime.Now.AddDays(-5) },
            new Order { Id = 2, CustomerId = 1, Amount = 200, Date = DateTime.Now.AddDays(-3) },
            new Order { Id = 3, CustomerId = 2, Amount = 150, Date = DateTime.Now.AddDays(-1) }
        };
    }
    
    public void DemonstrateBasicLinq()
    {
        // Query syntax
        var querySyntax = from c in customers
                          where c.City == "New York"
                          select c.Name;
        
        // Method syntax (lambda)
        var methodSyntax = customers
            .Where(c => c.City == "New York")
            .Select(c => c.Name);
        
        // Deferred execution
        var deferredQuery = customers.Where(c => c.City == "New York");
        
        customers.Add(new Customer { Id = 4, Name = "David", City = "New York" });
        
        // This will include David because query executes when enumerated
        foreach (var customer in deferredQuery)
        {
            Console.WriteLine(customer.Name);
        }
        
        // Immediate execution
        var immediateResult = customers
            .Where(c => c.City == "New York")
            .ToList();  // Executes immediately
    }
    
    public void DemonstrateLinqOperators()
    {
        // Select - projection
        var names = customers.Select(c => c.Name);
        
        // SelectMany - flatten nested collections
        var orderItems = customers.SelectMany(c => 
            orders.Where(o => o.CustomerId == c.Id)
                  .Select(o => new { CustomerName = c.Name, o.Amount }));
        
        // Where - filtering
        var highValueOrders = orders.Where(o => o.Amount > 100);
        
        // OfType - filter by type
        var numbers = new List<object> { 1, "two", 3, "four", 5 };
        var onlyIntegers = numbers.OfType<int>();
        
        // Set operators
        var list1 = new List<int> { 1, 2, 3, 4 };
        var list2 = new List<int> { 3, 4, 5, 6 };
        
        var distinct = list1.Distinct();
        var except = list1.Except(list2);    // { 1, 2 }
        var intersect = list1.Intersect(list2); // { 3, 4 }
        var union = list1.Union(list2);      // { 1, 2, 3, 4, 5, 6 }
        var concat = list1.Concat(list2);    // { 1, 2, 3, 4, 3, 4, 5, 6 }
        
        // Aggregation
        var totalAmount = orders.Sum(o => o.Amount);
        var averageAmount = orders.Average(o => o.Amount);
        var maxAmount = orders.Max(o => o.Amount);
        var minAmount = orders.Min(o => o.Amount);
        var orderCount = orders.Count();
        
        // Custom aggregation
        var product = numbers.OfType<int>().Aggregate((acc, x) => acc * x);
        
        string[] words = { "Hello", "World", "from", "LINQ" };
        string sentence = words.Aggregate((current, next) => current + " " + next);
    }
    
    public void DemonstrateSorting()
    {
        // OrderBy / OrderByDescending
        var sortedByName = customers.OrderBy(c => c.Name);
        var sortedByAmountDesc = orders.OrderByDescending(o => o.Amount);
        
        // ThenBy / ThenByDescending
        var sortedByCityThenName = customers
            .OrderBy(c => c.City)
            .ThenBy(c => c.Name);
        
        // Reverse
        var reversed = customers.Reverse();
    }
    
    public void DemonstrateGrouping()
    {
        // GroupBy
        var groupedByCity = customers.GroupBy(c => c.City);
        
        foreach (var group in groupedByCity)
        {
            Console.WriteLine($"City: {group.Key}");
            foreach (var customer in group)
            {
                Console.WriteLine($"  - {customer.Name}");
            }
        }
        
        // GroupBy with multiple keys
        var groupedByMultipleKeys = orders.GroupBy(o => new 
        { 
            o.CustomerId, 
            Month = o.Date.Month 
        });
        
        // ToLookup - immediate execution
        var lookup = customers.ToLookup(c => c.City);
        var newYorkCustomers = lookup["New York"];
    }
    
    public void DemonstrateJoins()
    {
        // Inner Join
        var innerJoin = from c in customers
                        join o in orders on c.Id equals o.CustomerId
                        select new { c.Name, o.Amount, o.Date };
        
        // Method syntax
        var innerJoinMethod = customers.Join(
            orders,
            c => c.Id,
            o => o.CustomerId,
            (c, o) => new { c.Name, o.Amount, o.Date });
        
        // Group Join (Left Join)
        var groupJoin = from c in customers
                        join o in orders on c.Id equals o.CustomerId into customerOrders
                        select new 
                        { 
                            c.Name, 
                            Orders = customerOrders,
                            TotalAmount = customerOrders.Sum(o => o.Amount)
                        };
        
        // Left Outer Join
        var leftJoin = from c in customers
                       join o in orders on c.Id equals o.CustomerId into customerOrders
                       from co in customerOrders.DefaultIfEmpty()
                       select new 
                       { 
                           c.Name, 
                           Amount = co?.Amount ?? 0 
                       };
        
        // Cross Join
        var crossJoin = from c in customers
                        from o in orders
                        select new { c.Name, o.Amount };
    }
    
    public void DemonstratePartitioning()
    {
        // Take / TakeWhile
        var first3 = customers.Take(3);
        var takeWhile = customers.TakeWhile(c => c.City == "New York");
        
        // Skip / SkipWhile
        var skip2 = customers.Skip(2);
        var skipWhile = customers.SkipWhile(c => c.City == "New York");
        
        // Paging
        int pageSize = 10;
        int pageNumber = 1;
        
        var pagedData = customers
            .Skip((pageNumber - 1) * pageSize)
            .Take(pageSize)
            .ToList();
    }
    
    public void DemonstrateElementOperators()
    {
        var first = customers.First();
        var firstOrDefault = customers.FirstOrDefault();
        var last = customers.Last();
        var single = customers.Single(c => c.Id == 1);
        var singleOrDefault = customers.SingleOrDefault(c => c.Id == 99);
        
        var elementAt = customers.ElementAt(1);
        var elementAtOrDefault = customers.ElementAtOrDefault(10);
        
        // SequenceEqual
        bool areEqual = customers.SequenceEqual(customers);
    }
    
    public void DemonstrateIQueryable()
    {
        // IEnumerable vs IQueryable
        // IEnumerable<T> - in-memory operations
        IEnumerable<Customer> enumerable = customers.Where(c => c.City == "New York");
        
        // IQueryable<T> - database operations (with EF Core)
        // IQueryable<Customer> queryable = dbContext.Customers.Where(c => c.City == "New York");
        
        // Expression trees (built automatically by IQueryable)
        // The lambda expression is converted to an expression tree
        // and then translated to SQL
    }
}

public class Customer
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string City { get; set; }
}

public class Order
{
    public int Id { get; set; }
    public int CustomerId { get; set; }
    public decimal Amount { get; set; }
    public DateTime Date { get; set; }
}
```

## **Chapter 20: Entity Framework Core**

```csharp
// Entity classes
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
    public int CategoryId { get; set; }
    public Category Category { get; set; }
    public bool IsDeleted { get; set; } // For soft delete
}

public class Category
{
    public int Id { get; set; }
    public string Name { get; set; }
    public ICollection<Product> Products { get; set; }
}

// DbContext
public class ApplicationDbContext : DbContext
{
    public DbSet<Product> Products { get; set; }
    public DbSet<Category> Categories { get; set; }
    
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
    }
    
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // Fluent API configuration
        modelBuilder.Entity<Product>(entity =>
        {
            entity.ToTable("Products");
            entity.HasKey(p => p.Id);
            
            entity.Property(p => p.Name)
                .IsRequired()
                .HasMaxLength(100);
            
            entity.Property(p => p.Price)
                .HasPrecision(18, 2);
            
            // Relationship
            entity.HasOne(p => p.Category)
                .WithMany(c => c.Products)
                .HasForeignKey(p => p.CategoryId)
                .OnDelete(DeleteBehavior.Restrict);
            
            // Index
            entity.HasIndex(p => p.Name);
            
            // Global query filter for soft delete
            entity.HasQueryFilter(p => !p.IsDeleted);
        });
        
        modelBuilder.Entity<Category>(entity =>
        {
            entity.HasKey(c => c.Id);
            entity.Property(c => c.Name).IsRequired();
            
            // Seed data
            entity.HasData(
                new Category { Id = 1, Name = "Electronics" },
                new Category { Id = 2, Name = "Books" }
            );
        });
    }
}

// Repository pattern with EF Core
public class Repository<T> : IRepository<T> where T : class
{
    protected readonly ApplicationDbContext _context;
    protected readonly DbSet<T> _dbSet;
    
    public Repository(ApplicationDbContext context)
    {
        _context = context;
        _dbSet = context.Set<T>();
    }
    
    public async Task<T> GetByIdAsync(int id)
    {
        return await _dbSet.FindAsync(id);
    }
    
    public async Task<IEnumerable<T>> GetAllAsync()
    {
        return await _dbSet.ToListAsync();
    }
    
    public async Task AddAsync(T entity)
    {
        await _dbSet.AddAsync(entity);
    }
    
    public void Update(T entity)
    {
        _dbSet.Update(entity);
    }
    
    public void Delete(T entity)
    {
        _dbSet.Remove(entity);
    }
    
    public async Task SaveChangesAsync()
    {
        await _context.SaveChangesAsync();
    }
}

// Service class demonstrating EF Core features
public class ProductService
{
    private readonly IRepository<Product> _productRepository;
    private readonly ApplicationDbContext _context;
    
    public ProductService(
        IRepository<Product> productRepository,
        ApplicationDbContext context)
    {
        _productRepository = productRepository;
        _context = context;
    }
    
    // Eager Loading
    public async Task<List<Product>> GetProductsWithCategoryAsync()
    {
        return await _context.Products
            .Include(p => p.Category)
            .ToListAsync();
    }
    
    // Multiple levels of eager loading
    public async Task<List<Product>> GetProductsWithNestedDataAsync()
    {
        return await _context.Products
            .Include(p => p.Category)
                .ThenInclude(/* further navigation properties */)
            .ToListAsync();
    }
    
    // Explicit Loading
    public async Task LoadCategoryExplicitly(Product product)
    {
        await _context.Entry(product)
            .Reference(p => p.Category)
            .LoadAsync();
    }
    
    public async Task LoadProductsExplicitly(Category category)
    {
        await _context.Entry(category)
            .Collection(c => c.Products)
            .LoadAsync();
    }
    
    // Lazy Loading (requires virtual navigation properties)
    // public class Product
    // {
    //     public virtual Category Category { get; set; }
    // }
    
    // Projection for performance
    public async Task<List<ProductDto>> GetProductDtosAsync()
    {
        return await _context.Products
            .Select(p => new ProductDto
            {
                Id = p.Id,
                Name = p.Name,
                Price = p.Price,
                CategoryName = p.Category.Name
            })
            .ToListAsync();
    }
    
    // Pagination
    public async Task<PagedResult<ProductDto>> GetPagedProductsAsync(
        int pageNumber, int pageSize, string searchTerm = null)
    {
        var query = _context.Products.AsQueryable();
        
        if (!string.IsNullOrEmpty(searchTerm))
        {
            query = query.Where(p => p.Name.Contains(searchTerm));
        }
        
        int totalCount = await query.CountAsync();
        
        var items = await query
            .Skip((pageNumber - 1) * pageSize)
            .Take(pageSize)
            .Select(p => new ProductDto
            {
                Id = p.Id,
                Name = p.Name,
                Price = p.Price
            })
            .ToListAsync();
        
        return new PagedResult<ProductDto>
        {
            Items = items,
            TotalCount = totalCount,
            PageNumber = pageNumber,
            PageSize = pageSize
        };
    }
    
    // Transactions
    public async Task TransferProductCategory(
        int productId, int newCategoryId)
    {
        using (var transaction = await _context.Database.BeginTransactionAsync())
        {
            try
            {
                var product = await _context.Products.FindAsync(productId);
                product.CategoryId = newCategoryId;
                await _context.SaveChangesAsync();
                
                await transaction.CommitAsync();
            }
            catch
            {
                await transaction.RollbackAsync();
                throw;
            }
        }
    }
    
    // Soft delete
    public async Task SoftDeleteProduct(int productId)
    {
        var product = await _context.Products.FindAsync(productId);
        if (product != null)
        {
            product.IsDeleted = true;
            await _context.SaveChangesAsync();
        }
    }
    
    // Disconnected entities
    public async Task UpdateDisconnectedProduct(Product updatedProduct)
    {
        var existingProduct = await _context.Products
            .AsNoTracking()
            .FirstOrDefaultAsync(p => p.Id == updatedProduct.Id);
        
        if (existingProduct == null)
        {
            throw new Exception("Product not found");
        }
        
        // Attach and mark as modified
        _context.Entry(updatedProduct).State = EntityState.Modified;
        await _context.SaveChangesAsync();
    }
    
    // Raw SQL
    public async Task<List<Product>> GetExpensiveProductsAsync()
    {
        decimal minPrice = 100;
        
        return await _context.Products
            .FromSqlInterpolated($"SELECT * FROM Products WHERE Price > {minPrice}")
            .ToListAsync();
    }
    
    // Stored procedure
    public async Task<List<Product>> GetProductsByCategoryAsync(int categoryId)
    {
        return await _context.Products
            .FromSqlInterpolated($"EXEC GetProductsByCategory @CategoryId = {categoryId}")
            .ToListAsync();
    }
    
    // Shadow properties
    // Configured in OnModelCreating:
    // modelBuilder.Entity<Product>().Property<DateTime>("LastModified");
    
    public async Task UpdateWithShadowProperty(int productId)
    {
        var product = await _context.Products.FindAsync(productId);
        _context.Entry(product).Property("LastModified").CurrentValue = DateTime.UtcNow;
        await _context.SaveChangesAsync();
    }
    
    // Concurrency handling
    // Add to entity: public byte[] RowVersion { get; set; }
    // Configure: entity.Property(p => p.RowVersion).IsRowVersion();
    
    public async Task UpdateWithConcurrencyCheck(Product updatedProduct)
    {
        try
        {
            _context.Entry(updatedProduct).State = EntityState.Modified;
            await _context.SaveChangesAsync();
        }
        catch (DbUpdateConcurrencyException ex)
        {
            // Handle concurrency conflict
            foreach (var entry in ex.Entries)
            {
                var databaseValues = await entry.GetDatabaseValuesAsync();
                // Resolve conflict (e.g., client wins, database wins, merge)
                entry.OriginalValues.SetValues(databaseValues);
            }
        }
    }
    
    // Inheritance mapping (Table-per-Hierarchy)
    public class DigitalProduct : Product
    {
        public string DownloadUrl { get; set; }
        public long FileSize { get; set; }
    }
    
    public class PhysicalProduct : Product
    {
        public decimal Weight { get; set; }
        public string Dimensions { get; set; }
    }
}

public class ProductDto
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
    public string CategoryName { get; set; }
}

public class PagedResult<T>
{
    public List<T> Items { get; set; }
    public int TotalCount { get; set; }
    public int PageNumber { get; set; }
    public int PageSize { get; set; }
    public int TotalPages => (int)Math.Ceiling(TotalCount / (double)PageSize);
}

public interface IRepository<T> where T : class
{
    Task<T> GetByIdAsync(int id);
    Task<IEnumerable<T>> GetAllAsync();
    Task AddAsync(T entity);
    void Update(T entity);
    void Delete(T entity);
    Task SaveChangesAsync();
}
```

## **Chapter 27: SOLID Design Principles**

```csharp
// 1. Single Responsibility Principle (SRP)
// A class should have only one reason to change

// Bad example
public class Employee
{
    public void CalculatePay() { /* Payroll logic */ }
    public void Save() { /* Database logic */ }
    public void GenerateReport() { /* Reporting logic */ }
}

// Good example
public class Employee
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Salary { get; set; }
}

public class PayrollCalculator
{
    public decimal CalculatePay(Employee employee)
    {
        // Payroll calculation logic
        return employee.Salary / 12;
    }
}

public class EmployeeRepository
{
    public void Save(Employee employee)
    {
        // Database persistence logic
    }
}

public class EmployeeReportGenerator
{
    public void GenerateReport(Employee employee)
    {
        // Report generation logic
    }
}

// 2. Open-Closed Principle (OCP)
// Classes should be open for extension but closed for modification

// Bad example
public class AreaCalculator
{
    public double CalculateArea(object shape)
    {
        if (shape is Rectangle)
        {
            var rect = (Rectangle)shape;
            return rect.Width * rect.Height;
        }
        else if (shape is Circle)
        {
            var circle = (Circle)shape;
            return Math.PI * circle.Radius * circle.Radius;
        }
        // Must modify this class to add new shapes
        return 0;
    }
}

// Good example
public interface IShape
{
    double CalculateArea();
}

public class Rectangle : IShape
{
    public double Width { get; set; }
    public double Height { get; set; }
    
    public double CalculateArea()
    {
        return Width * Height;
    }
}

public class Circle : IShape
{
    public double Radius { get; set; }
    
    public double CalculateArea()
    {
        return Math.PI * Radius * Radius;
    }
}

public class AreaCalculator
{
    public double CalculateArea(IShape shape)
    {
        return shape.CalculateArea();
    }
}

// 3. Liskov Substitution Principle (LSP)
// Derived classes must be substitutable for their base classes

// Bad example
public class Bird
{
    public virtual void Fly()
    {
        Console.WriteLine("Flying");
    }
}

public class Penguin : Bird
{
    public override void Fly()
    {
        throw new NotSupportedException("Penguins can't fly!");
    }
}

// Good example
public interface IFlyable
{
    void Fly();
}

public interface ISwimmable
{
    void Swim();
}

public class Sparrow : IFlyable
{
    public void Fly()
    {
        Console.WriteLine("Sparrow flying");
    }
}

public class Penguin : ISwimmable
{
    public void Swim()
    {
        Console.WriteLine("Penguin swimming");
    }
}

// 4. Interface Segregation Principle (ISP)
// Clients should not be forced to depend on interfaces they don't use

// Bad example
public interface IWorker
{
    void Work();
    void Eat();
    void Sleep();
}

// Good example
public interface IWorkable
{
    void Work();
}

public interface IEatable
{
    void Eat();
}

public interface ISleepable
{
    void Sleep();
}

public class Human : IWorkable, IEatable, ISleepable
{
    public void Work() { Console.WriteLine("Human working"); }
    public void Eat() { Console.WriteLine("Human eating"); }
    public void Sleep() { Console.WriteLine("Human sleeping"); }
}

public class Robot : IWorkable
{
    public void Work() { Console.WriteLine("Robot working"); }
    // No need to implement Eat or Sleep
}

// 5. Dependency Inversion Principle (DIP)
// High-level modules should not depend on low-level modules.
// Both should depend on abstractions.

// Bad example
public class EmailNotifier
{
    public void SendEmail(string message)
    {
        Console.WriteLine($"Sending email: {message}");
    }
}

public class NotificationService
{
    private EmailNotifier _emailNotifier = new EmailNotifier();
    
    public void Notify(string message)
    {
        _emailNotifier.SendEmail(message);
    }
}

// Good example
public interface INotifier
{
    void Send(string message);
}

public class EmailNotifier : INotifier
{
    public void Send(string message)
    {
        Console.WriteLine($"Sending email: {message}");
    }
}

public class SmsNotifier : INotifier
{
    public void Send(string message)
    {
        Console.WriteLine($"Sending SMS: {message}");
    }
}

public class NotificationService
{
    private readonly INotifier _notifier;
    
    public NotificationService(INotifier notifier)
    {
        _notifier = notifier;
    }
    
    public void Notify(string message)
    {
        _notifier.Send(message);
    }
}

// Usage with Dependency Injection
var emailService = new NotificationService(new EmailNotifier());
var smsService = new NotificationService(new SmsNotifier());
```

## **Chapter 32: Dependency Injection**

```csharp
// Service interfaces
public interface ILogger
{
    void Log(string message);
}

public interface IDataService
{
    Task<string> GetDataAsync();
}

// Service implementations
public class ConsoleLogger : ILogger
{
    public void Log(string message)
    {
        Console.WriteLine($"[{DateTime.Now}] {message}");
    }
}

public class FileLogger : ILogger
{
    private readonly string _filePath;
    
    public FileLogger(string filePath)
    {
        _filePath = filePath;
    }
    
    public void Log(string message)
    {
        File.AppendAllText(_filePath, $"[{DateTime.Now}] {message}\n");
    }
}

public class DataService : IDataService
{
    private readonly ILogger _logger;
    
    public DataService(ILogger logger)
    {
        _logger = logger;
    }
    
    public async Task<string> GetDataAsync()
    {
        _logger.Log("Fetching data...");
        await Task.Delay(100);
        _logger.Log("Data fetched successfully");
        return "Sample Data";
    }
}

// DI Container setup (using Microsoft.Extensions.DependencyInjection)
public class DependencyInjectionDemo
{
    public ServiceProvider ConfigureServices()
    {
        var services = new ServiceCollection();
        
        // Singleton - same instance throughout app lifetime
        services.AddSingleton<ILogger>(provider =>
        {
            return new ConsoleLogger();
        });
        
        // Scoped - same instance within a scope (e.g., HTTP request)
        services.AddScoped<IDataService, DataService>();
        
        // Transient - new instance every time
        services.AddTransient<SomeService>();
        
        // Factory pattern
        services.AddSingleton<Func<string, ILogger>>(provider =>
        {
            return (filePath) => new FileLogger(filePath);
        });
        
        return services.BuildServiceProvider();
    }
    
    public void DemonstrateDI()
    {
        var serviceProvider = ConfigureServices();
        
        // Resolve service
        var dataService = serviceProvider.GetRequiredService<IDataService>();
        
        // Create scope
        using (var scope = serviceProvider.CreateScope())
        {
            var scopedService = scope.ServiceProvider.GetRequiredService<IDataService>();
        }
    }
}

// Property Injection pattern
public class PropertyInjectionExample
{
    public ILogger Logger { get; set; }  // Property injection
    
    public void DoWork()
    {
        Logger?.Log("Work started");
    }
}

// Method Injection pattern
public class MethodInjectionExample
{
    public void ProcessData(string data, ILogger logger)  // Method injection
    {
        logger.Log($"Processing: {data}");
    }
}

// Custom DI container (simplified)
public class SimpleContainer
{
    private readonly Dictionary<Type, Type> _registrations = new();
    private readonly Dictionary<Type, object> _singletons = new();
    
    public void Register<TInterface, TImplementation>() 
        where TImplementation : TInterface
    {
        _registrations[typeof(TInterface)] = typeof(TImplementation);
    }
    
    public void RegisterSingleton<TInterface>(TInterface instance)
    {
        _singletons[typeof(TInterface)] = instance;
    }
    
    public T Resolve<T>()
    {
        if (_singletons.TryGetValue(typeof(T), out var singleton))
        {
            return (T)singleton;
        }
        
        if (_registrations.TryGetValue(typeof(T), out var implementationType))
        {
            var constructor = implementationType.GetConstructors()
                .First();
            
            var parameters = constructor.GetParameters()
                .Select(p => Resolve(p.ParameterType))
                .ToArray();
            
            var instance = Activator.CreateInstance(
                implementationType, parameters);
            
            return (T)instance;
        }
        
        throw new InvalidOperationException(
            $"Type {typeof(T)} not registered");
    }
    
    private object Resolve(Type type)
    {
        var method = typeof(SimpleContainer)
            .GetMethod(nameof(Resolve))
            .MakeGenericMethod(type);
        
        return method.Invoke(this, null);
    }
}
```

## **Chapter 34: New Features in C#**

```csharp
public class CSharpNewFeatures
{
    // C# 7.0+ Features
    
    // Out variables
    public void OutVariableExample(string input)
    {
        // Old way
        int result;
        if (int.TryParse(input, out result))
        {
            Console.WriteLine(result);
        }
        
        // New way - inline declaration
        if (int.TryParse(input, out int parsedValue))
        {
            Console.WriteLine(parsedValue);
        }
    }
    
    // Pattern Matching
    public decimal CalculateDiscount(Product product)
    {
        // Type pattern
        if (product is DigitalProduct digitalProduct)
        {
            return digitalProduct.Price * 0.1m;
        }
        
        // Switch pattern matching
        decimal discount = product switch
        {
            DigitalProduct { Price: > 100 } => 0.15m,
            DigitalProduct => 0.10m,
            PhysicalProduct { Price: > 50 } => 0.05m,
            _ => 0m
        };
        
        return discount;
    }
    
    // Property pattern
    public bool IsExpensive(Product product)
    {
        return product is { Price: > 100 };
    }
    
    // Tuple patterns
    public string RockPaperScissors(string first, string second)
    {
        return (first, second) switch
        {
            ("rock", "paper") => "Paper wins",
            ("rock", "scissors") => "Rock wins",
            ("paper", "rock") => "Paper wins",
            ("paper", "scissors") => "Scissors wins",
            ("scissors", "rock") => "Rock wins",
            ("scissors", "paper") => "Scissors wins",
            (_, _) => "Tie"
        };
    }
    
    // Tuples and Deconstruction
    public (string FirstName, string LastName) GetFullName()
    {
        return ("John", "Doe");
    }
    
    public void TupleDeconstruction()
    {
        var (firstName, lastName) = GetFullName();
        Console.WriteLine($"{firstName} {lastName}");
        
        // With discard
        var (name, _) = GetFullName();
    }
    
    // Local functions
    public int Calculate(int x, int y)
    {
        return Add(x, y);
        
        // Local function
        int Add(int a, int b)
        {
            return a + b;
        }
    }
    
    // Ref returns and ref locals
    public ref int Find(int[] numbers, int target)
    {
        for (int i = 0; i < numbers.Length; i++)
        {
            if (numbers[i] == target)
                return ref numbers[i];  // Return reference
        }
        throw new IndexOutOfRangeException();
    }
    
    public void RefLocalExample()
    {
        int[] numbers = { 1, 2, 3, 4, 5 };
        ref int numRef = ref Find(numbers, 3);
        numRef = 10;  // Modifies the actual array element
    }
    
    // Expression-bodied members
    public class ExpressionBodiedExample
    {
        private string name;
        
        // Constructor
        public ExpressionBodiedExample(string name) => this.name = name;
        
        // Destructor
        ~ExpressionBodiedExample() => Console.WriteLine("Finalized");
        
        // Property
        public string Name
        {
            get => name;
            set => name = value ?? throw new ArgumentNullException(nameof(value));
        }
        
        // Read-only property
        public string Greeting => $"Hello, {name}";
        
        // Method
        public override string ToString() => $"Name: {name}";
    }
    
    // Async Main (C# 7.1)
    static async Task Main(string[] args)
    {
        await Task.Delay(100);
        Console.WriteLine("Async Main!");
    }
    
    // Default interface methods (C# 8)
    public interface IModernLogger
    {
        void Log(string message);
        
        // Default implementation
        void LogError(string error)
        {
            Log($"[ERROR] {error}");
        }
        
        // Static method in interface (C# 8+)
        static IModernLogger CreateDefault()
        {
            return new ConsoleLogger();
        }
    }
    
    // Nullable reference types (C# 8)
    #nullable enable
    public class NullableReferenceExample
    {
        public string NonNullable { get; set; }  // Must be initialized
        public string? Nullable { get; set; }    // Can be null
        
        public void Process(string? input)
        {
            // Null check required
            int length = input?.Length ?? 0;
            
            // Null-forgiving operator
            string definitelyNotNull = input!;
        }
    }
    
    // Indices and ranges (C# 8)
    public void IndexAndRangeExample()
    {
        int[] numbers = { 0, 1, 2, 3, 4, 5 };
        
        int last = numbers[^1];    // 5
        int secondLast = numbers[^2]; // 4
        
        int[] slice1 = numbers[1..3];   // { 1, 2 }
        int[] slice2 = numbers[..3];    // { 0, 1, 2 }
        int[] slice3 = numbers[3..];    // { 3, 4, 5 }
        int[] slice4 = numbers[^2..];   // { 4, 5 }
        
        Range range = 1..4;
        int[] slice5 = numbers[range];
    }
    
    // Null-coalescing assignment (C# 8)
    public void NullCoalescingAssignmentExample()
    {
        string? value = null;
        value ??= "Default Value";
        Console.WriteLine(value);  // "Default Value"
    }
}

// C# 9.0 Features

// Record types
public record Person(string FirstName, string LastName);

public record Employee : Person
{
    public int EmployeeId { get; init; }
    public decimal Salary { get; init; }
    
    public Employee(string firstName, string lastName, int employeeId, decimal salary)
        : base(firstName, lastName)
    {
        EmployeeId = employeeId;
        Salary = salary;
    }
}

public class RecordExamples
{
    public void DemonstrateRecords()
    {
        var person1 = new Person("John", "Doe");
        var person2 = new Person("John", "Doe");
        
        // Value-based equality
        Console.WriteLine(person1 == person2);  // True
        
        // Immutability
        var person3 = person1 with { LastName = "Smith" };
        
        // Deconstruction
        var (firstName, lastName) = person1;
    }
}

// C# 10.0 Features

// File-scoped namespaces
namespace MyApplication.Features;

// Global usings (in a separate file, e.g., GlobalUsings.cs)
// global using System;
// global using System.Collections.Generic;

// Record structs
public readonly record struct Point(int X, int Y);

// C# 12.0 Features

// Primary constructors for non-record classes
public class BankAccount(string accountId, string ownerName)
{
    public string AccountId { get; } = accountId;
    public string OwnerName { get; set; } = ownerName;
    public decimal Balance { get; private set; }
    
    public void Deposit(decimal amount) => Balance += amount;
}

// Collection literals
public class CollectionLiteralExample
{
    public void Demonstrate()
    {
        int[] array = [1, 2, 3, 4, 5];
        List<string> list = ["one", "two", "three"];
        Span<int> span = [1, 2, 3];
    }
}

// Default values for lambda parameters
public class LambdaDefaultParameters
{
    private Func<int, int, int> multiply = (x, y = 2) => x * y;
    private Action<string> log = (message = "Default") => Console.WriteLine(message);
}

// Inline arrays
[System.Runtime.CompilerServices.InlineArray(10)]
public struct Buffer<T>
{
    private T _element0;
}

// Enhanced switch expressions
public class EnhancedSwitch
{
    public string GetDayType(DayOfWeek day) => day switch
    {
        DayOfWeek.Saturday or DayOfWeek.Sunday => "Weekend",
        _ => "Weekday"
    };
}
```

This comprehensive guide covers the major topics from your C# curriculum. Each section includes practical code examples that demonstrate the concepts in action. To master C#:

1. **Practice each concept** - Type out the examples and modify them
2. **Build projects** - Create real applications using these concepts
3. **Debug thoroughly** - Use breakpoints to understand execution flow
4. **Read documentation** - Refer to Microsoft's official C# documentation
5. **Join communities** - Stack Overflow, GitHub, and C# Discord servers are great resources
6. **Write unit tests** - Test your code to ensure it works correctly
7. **Review and refactor** - Continuously improve your code quality

