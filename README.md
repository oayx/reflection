
## 用法举例

```cpp

//---------- 示例类 ----------
class UActor : public UObject {
	GENERATED_BODY(UActor, UObject)
};


class UPawn : public UActor {
	GENERATED_BODY(UPawn, UActor)

public:
	// 属性声明
	UPROPERTY(int, Health, EditAnywhere, Category = Gameplay | Attributes, DisplayName = 生命值)
	int Health = 100;

	UPROPERTY(float, MoveSpeed, BlueprintReadWrite, Category = Movement)
	float MoveSpeed = 5.0f;

	UPROPERTY(std::string, Name, AdvancedDisplay)
	std::string Name = "Pawn";

	// 无参数函数
	UFUNCTION(void, TakeDamage)
	void TakeDamage() {
		std::cout << Name << " takes 10 damage!\n";
		Health -= 10;
		if (Health <= 0) {
			std::cout << Name << " has been destroyed!\n";
		}
	}

	// 带参数函数
	UFUNCTION(void, Heal, int amount)
	void Heal(int amount) {
		std::cout << Name << " heals " << amount << " HP!\n";
		Health += amount;
		if (Health > 100) Health = 100;
	}

	// 带返回值的函数
	UFUNCTION(bool, CanAttack, float range)
	bool CanAttack(float range) {
		std::cout << "Checking attack range: " << range << "\n";
		return range <= 10.0f;
	}

	// 带参数和返回值的函数
	UFUNCTION(float, CalculateDamage, float baseDamage, float multiplier)
	float CalculateDamage(float baseDamage, float multiplier) {
		std::cout << "Calculating damage: " << baseDamage << " * " << multiplier << "\n";
		return baseDamage * multiplier;
	}
};

//---------- 测试代码 ----------
void PrintMetaInfo(const Property& prop) {
	std::cout << "  " << prop.name << " [" << prop.type << "]\n";
	std::cout << "  Metadata:\n";

	if (prop.meta.HasFlag("EditAnywhere")) {
		std::cout << "    - Editable\n";
	}

	if (auto category = prop.meta.GetValue("Category"); !category.empty()) {
		std::cout << "    - Category: " << category << "\n";
	}

	if (auto displayName = prop.meta.GetValue("DisplayName"); !displayName.empty()) {
		std::cout << "    - Display Name: " << displayName << "\n";
	}
}

void PrintFunctionInfo(const FunctionInfo& func) {
	std::cout << "  " << func.name << "() -> " << func.returnType << "\n";
	std::cout << "  Parameters: ";
	for (const auto& param : func.paramTypes) {
		std::cout << param << " ";
	}
	std::cout << "\n";

	std::cout << "  Metadata:\n";
	if (func.meta.HasFlag("BlueprintCallable")) {
		std::cout << "    - Blueprint Callable\n";
	}
}

int main() {
	UPawn pawn;
	pawn.Name = "Hero";

	std::cout << "Class: " << pawn.GetClass()->name << "\n";

	std::cout << "\nProperties:\n";
	for (const auto& prop : pawn.GetClass()->properties) {
		PrintMetaInfo(prop);
		std::cout << "----------\n";
	}

	std::cout << "\nFunctions:\n";
	for (const auto& func : pawn.GetClass()->functions) {
		PrintFunctionInfo(*func);
		std::cout << "----------\n";
	}

	// 直接调用函数
	std::cout << "\nDirect function calls:\n";
	pawn.TakeDamage();
	pawn.Heal(20);
	std::cout << "Can attack at 15m: " << std::boolalpha << pawn.CanAttack(15.0f) << "\n";
	std::cout << "Damage: " << pawn.CalculateDamage(50.0f, 1.5f) << "\n";

	// 反射调用函数
	std::cout << "\nReflection function calls:\n";
	pawn.Invoke("TakeDamage");
	pawn.Invoke("Heal", 15);
	std::cout << "Can attack at 8m: "
		<< pawn.Invoke<bool>("CanAttack", 8.0f) << "\n";
	std::cout << "Damage: "
		<< pawn.Invoke<float>("CalculateDamage", 75.0f, 2.0f) << "\n";

	return 0;
}
```
