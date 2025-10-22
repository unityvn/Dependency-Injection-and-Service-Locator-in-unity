# 🔄 So sánh Dependency Injection và Service Locator trong Unity

## 🧩 1. Dependency Injection (DI)
### 📘 Định nghĩa
Dependency Injection là một kỹ thuật thiết kế trong đó các phụ thuộc (dependencies) của một đối tượng được cung cấp từ bên ngoài thay vì được khởi tạo bên trong đối tượng đó.

### ✅ Ưu điểm
- **Tăng tính module hóa**: Các lớp không phụ thuộc trực tiếp vào việc khởi tạo các phụ thuộc, giúp dễ dàng thay thế hoặc mở rộng.
- **Dễ kiểm thử**: DI giúp dễ dàng mock các phụ thuộc trong unit test.
- **Tuân thủ nguyên tắc SOLID**: Đặc biệt là nguyên tắc Inversion of Control (IoC).

### ⚠️ Nhược điểm
- **Phức tạp hơn**: Cần thiết lập các container DI hoặc framework DI (như Zenject trong Unity).
- **Hiệu năng**: Có thể ảnh hưởng đến hiệu năng nếu không được tối ưu hóa.

### 🛠️ Ứng dụng trong Unity
- Sử dụng các framework như Zenject, Extenject để quản lý DI.
- Thích hợp cho các dự án lớn, nơi cần quản lý nhiều phụ thuộc phức tạp.

### 💡 Ví dụ triển khai
```csharp
// filepath: Assets/Scripts/DIExample.cs
using UnityEngine;

// Interface cho dịch vụ
public interface IWeapon
{
    void Attack();
}

// Triển khai dịch vụ
public class Sword : IWeapon
{
    public void Attack()
    {
        Debug.Log("Attacking with a sword!");
    }
}

// Lớp sử dụng dịch vụ
public class Player
{
    private IWeapon _weapon;

    // Inject phụ thuộc thông qua constructor
    public Player(IWeapon weapon)
    {
        _weapon = weapon;
    }

    public void PerformAttack()
    {
        _weapon.Attack();
    }
}

// Sử dụng DI
public class Game : MonoBehaviour
{
    void Start()
    {
        // Inject phụ thuộc
        IWeapon weapon = new Sword();
        Player player = new Player(weapon);

        player.PerformAttack();
    }
}
```

> ℹ️ Ghi chú: Ví dụ trên minh họa DI trong C# thuần với constructor injection. Khi áp dụng cho `MonoBehaviour` trong Unity, thường cần kết hợp với các container DI (như Zenject) để inject vào vòng đời component.

---

## 🧭 2. Service Locator (SL)
### 📘 Định nghĩa
Service Locator là một mẫu thiết kế cung cấp một lớp trung gian (locator) để truy xuất các dịch vụ hoặc phụ thuộc cần thiết.

### ✅ Ưu điểm
- **Dễ triển khai**: Không cần sử dụng các framework bên ngoài.
- **Tập trung**: Tất cả các dịch vụ được quản lý tại một nơi duy nhất.
- **Hiệu năng tốt hơn**: So với DI, SL thường có hiệu năng tốt hơn trong các dự án nhỏ.

### ⚠️ Nhược điểm
- **Khó kiểm thử**: Do các phụ thuộc được truy xuất thông qua locator, việc mock các phụ thuộc trở nên khó khăn hơn.
- **Phụ thuộc chặt chẽ**: Các lớp phụ thuộc vào locator, vi phạm nguyên tắc Inversion of Control.
- **Khó mở rộng**: Khi số lượng dịch vụ tăng lên, locator có thể trở nên phức tạp.

### 🛠️ Ứng dụng trong Unity
- Phù hợp cho các dự án nhỏ hoặc các dự án không yêu cầu tính module hóa cao.
- Có thể tự xây dựng một Service Locator đơn giản hoặc sử dụng các giải pháp có sẵn.

### 💡 Ví dụ triển khai
```csharp
// filepath: Assets/Scripts/SLExample.cs
using UnityEngine;
using System.Collections.Generic;

// Interface cho dịch vụ
public interface IWeapon
{
    void Attack();
}

// Triển khai dịch vụ
public class Sword : IWeapon
{
    public void Attack()
    {
        Debug.Log("Attacking with a sword!");
    }
}

// Service Locator
public class ServiceLocator
{
    private static ServiceLocator _instance;
    public static ServiceLocator Instance => _instance ??= new ServiceLocator();

    private readonly Dictionary<System.Type, object> _services = new Dictionary<System.Type, object>();

    public void Register<T>(T service)
    {
        _services[typeof(T)] = service;
    }

    public T Get<T>()
    {
        if (_services.TryGetValue(typeof(T), out var service))
        {
            return (T)service;
        }

        throw new System.InvalidOperationException($"Service {typeof(T).Name} chưa được đăng ký trong ServiceLocator.");
    }
}

// Lớp sử dụng dịch vụ
public class Player : MonoBehaviour
{
    private IWeapon _weapon;

    void Start()
    {
        // Lấy dịch vụ từ Service Locator
        _weapon = ServiceLocator.Instance.Get<IWeapon>();
        _weapon.Attack();
    }
}

// Sử dụng Service Locator
public class Game : MonoBehaviour
{
    void Awake()
    {
        // Đăng ký dịch vụ
        ServiceLocator.Instance.Register<IWeapon>(new Sword());
    }
}
```

---

## ⚖️ 3. So sánh

| Tiêu chí                | Dependency Injection (DI)         | Service Locator (SL)            |
|-------------------------|-----------------------------------|---------------------------------|
| **Mức độ module hóa**   | Cao                               | Thấp                            |
| **Dễ kiểm thử**         | Dễ dàng                           | Khó khăn                        |
| **Hiệu năng**           | Có thể chậm hơn                   | Tốt hơn                         |
| **Độ phức tạp triển khai** | Cao (cần framework hoặc container) | Thấp (có thể tự triển khai)     |
| **Tuân thủ SOLID**      | Tốt                               | Kém                             |
| **Ứng dụng**            | Dự án lớn, phức tạp               | Dự án nhỏ, đơn giản             |

---

## ✅ 4. Kết luận
- **Dependency Injection** phù hợp cho các dự án lớn, nơi cần quản lý nhiều phụ thuộc và ưu tiên tính module hóa, dễ kiểm thử.
- **Service Locator** phù hợp cho các dự án nhỏ, nơi hiệu năng và sự đơn giản được ưu tiên hơn.
