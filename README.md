# 🧠 Semantic Domain Architecture

Cursor rules that make your Pydantic models the single source of truth for business logic.

## Quick Start

```bash
# Install the rules
mkdir -p .cursor/rules
cp *.mdc .cursor/rules/

# That's it. Cursor will now use these rules when generating Python code.
```

**Using [Cursor](https://cursor.com)?** These rules help AI understand that your models should be intelligent domain objects, not anemic data bags.

**Not using Cursor?** Treat these as architecture guidelines for building semantic domain models.

## What You Get

Four rules that work together:

### [`000-semantic-domain-architecture.mdc`](000-semantic-domain-architecture.mdc)
Core philosophy - why models should contain business logic. Establishes the mental model.

### [`010-implementation-patterns.mdc`](010-implementation-patterns.mdc) 
Concrete patterns - value objects, domain models, enums with behavior, protocols, thin services.

### [`020-quick-decision-guide.mdc`](020-quick-decision-guide.mdc)
Decision flowchart - "I see a primitive type" → "Make it a value object". Quick reference while coding.

### [`030-complete-semantic-patterns.mdc`](030-complete-semantic-patterns.mdc)
Advanced patterns - discriminated unions, cross-field validation, computed fields, temporal modeling.

---

## Why This Exists

Every mature codebase eventually develops the same disease. Your domain knowledge—the rules that make your business unique—gets scattered across service classes, validators, utilities, and worse, implicit assumptions in UI code. Meanwhile, your domain models become hollow shells, mere data transfer objects that know nothing about the reality they represent.

```python
# Anemic model
class Order:
    items: List[dict]
    total: float
    status: str

# Logic scattered everywhere  
class OrderService:
    def calculate_total(self, order): ...
    def validate_status_change(self, order, new_status): ...
    def check_inventory(self, order): ...
    def apply_discounts(self, order): ...
```

This pattern emerged from N-tier architecture, where models couldn't have dependencies. But that constraint doesn't exist anymore. Modern frameworks like Pydantic give us models that can validate, compute, and transform. Yet we still write anemic models out of habit.

Semantic Domain Architecture breaks that habit. Your models become intelligent agents that understand their domain:

```python
# Semantic model
class Order(BaseModel):
    items: list[LineItem]
    status: OrderStatus
    
    model_config = {"frozen": True}
    
    @computed_field
    @property
    def total(self) -> Money:
        """Order calculates its own total."""
        return sum((item.total for item in self.items), Money.zero())
    
    def can_transition_to(self, new_status: OrderStatus) -> bool:
        """Order knows valid transitions."""
        return new_status in self.status.valid_transitions()
```

The shift is profound. Your models stop being passive data holders and become active participants that encode your business reality. Services shrink to mere orchestrators. Tests simplify because models validate themselves. And most importantly, your domain logic lives in one place—with the data it governs.

## Core Principles

### 1. No Naked Primitives

In physics, units matter. You can't add meters to seconds. Yet in code, we routinely use `float` for money, `string` for email, `int` for age. These "naked primitives" hide critical domain knowledge.

```python
# ❌ Bad
price: float
email: str
quantity: int

# ✅ Good  
price: Money  # Knows currency, precision, operations
email: Email  # Validates format, provides domain operations
quantity: Quantity  # Ensures positive, knows unit of measure
```

When you create a `Money` type, you're not just adding validation. You're encoding that money has currency, that different currencies can't be mixed, that monetary calculations need specific rounding rules. This knowledge belongs in your type system, not scattered across your codebase.

### 2. Behavior Lives with Data

Object-oriented programming promised encapsulation, but somewhere we forgot. We extract behavior into services, leaving models as property bags. This violates the fundamental principle: data and the operations on that data should live together.

```python
class Temperature(BaseModel):
    celsius: Decimal = Field(ge=-273.15)  # Absolute zero
    
    def to_fahrenheit(self) -> Decimal:
        return self.celsius * Decimal('1.8') + 32
    
    @computed_field
    @property
    def is_freezing(self) -> bool:
        return self.celsius <= 0
```

This model doesn't just store temperature—it understands temperature. It knows about absolute zero, unit conversions, and state transitions. When behavior lives with data, impossible operations become unrepresentable.

### 3. Immutable State Transitions

Mutable state is where bugs breed. When objects change underneath you, reasoning about program behavior becomes exponentially harder. Semantic models embrace immutability:

```python
def place_order(self) -> "Order":
    """Return new state instead of mutating."""
    if self.status != OrderStatus.DRAFT:
        raise ValueError("Can only place draft orders")
    
    return self.model_copy(update={
        "status": OrderStatus.PLACED,
        "placed_at": datetime.now()
    })
```

Each state transition returns a new instance. This isn't just functional programming dogma—it enables time-travel debugging, safe concurrent access, and clear audit trails. Your models become a sequence of immutable states rather than a mutable object with hidden history.

### 4. Smart Enums

Enums typically just name constants. But in your domain, those constants have relationships, rules, and behavior. Don't hide that knowledge in service classes:

```python
class OrderStatus(Enum):
    DRAFT = "draft"
    PLACED = "placed"
    SHIPPED = "shipped"
    
    def valid_transitions(self) -> set["OrderStatus"]:
        transitions = {
            self.DRAFT: {self.PLACED},
            self.PLACED: {self.SHIPPED},
            self.SHIPPED: set()
        }
        return transitions.get(self, set())
```

This enum doesn't just list statuses—it encodes your business workflow. The state machine lives where it belongs, making invalid transitions impossible to express.

## Real-World Example

Theory is nice, but let's see the transformation in practice. Here's a payment processing system, before and after:

```python
# Before: Logic in services
class PaymentService:
    def process_payment(self, payment_data: dict) -> dict:
        # Validate amount
        if payment_data['amount'] <= 0:
            raise ValueError("Invalid amount")
        
        # Calculate fees
        if payment_data['method'] == 'credit':
            fee = payment_data['amount'] * 0.029 + 0.30
        elif payment_data['method'] == 'ach':
            fee = 0.25
        # ... 200 more lines

# After: Logic in models  
class Payment(BaseModel):
    amount: Money
    method: PaymentMethod
    
    model_config = {"frozen": True}
    
    @field_validator('amount')
    @classmethod
    def validate_positive(cls, v: Money) -> Money:
        if v.amount <= 0:
            raise ValueError("Payment amount must be positive")
        return v
    
    @computed_field
    @property
    def processing_fee(self) -> Money:
        """Payment knows its own fees."""
        return self.method.calculate_fee(self.amount)
    
    def process(self, processor: PaymentProcessor) -> PaymentResult:
        """Payment knows how to process itself."""
        return processor.charge(
            amount=self.amount + self.processing_fee,
            method=self.method
        )
```

Notice what happened. The service's 200 lines of procedural logic became declarative model properties. Fee calculation moved to where it belongs—with the payment method. Validation became a field constraint, impossible to bypass. The model doesn't just hold payment data; it embodies payment processing knowledge.

## Advanced Patterns

Once you embrace semantic models, powerful patterns emerge that eliminate entire categories of bugs:

### Discriminated Unions (No More isinstance)

Traditional polymorphism in Python requires isinstance checks, visitor patterns, or abstract base classes. Discriminated unions offer a cleaner approach:

```python
from typing import Annotated, Discriminator

class EmailNotification(BaseModel):
    type: Literal["email"] = "email"
    to: Email
    subject: str

class SmsNotification(BaseModel):
    type: Literal["sms"] = "sms"
    to: PhoneNumber
    message: str

Notification = Annotated[Union[EmailNotification, SmsNotification], Discriminator('type')]

# Pydantic handles dispatch - no isinstance needed
def send(notification: Notification) -> None:
    notification.send()  # Each type has its own send() method
```

The discriminator field tells Pydantic which type to instantiate. No manual type checking, no visitor pattern, no match statements. The type system handles dispatch automatically.

### Cross-Field Validation

Real-world constraints often span multiple fields. Traditional validation scatters these rules across service methods. Pydantic's validation context keeps them with the model:

```python
class PriceRange(BaseModel):
    min_price: Money
    max_price: Money
    
    @field_validator('max_price')
    @classmethod
    def validate_range(cls, v: Money, info: ValidationInfo) -> Money:
        if min_price := info.data.get('min_price'):
            if v < min_price:
                raise ValueError("Max price must be >= min price")
        return v
```

The validator has access to all fields through `ValidationInfo`. Complex invariants become explicit constraints, impossible to violate regardless of how the model is constructed.

## Getting Started

The hardest part isn't learning these patterns—it's unlearning the service-oriented habits. Start small:

1. Pick your most complex service
2. Find the anemic model it operates on
3. Move one method from service to model
4. Run your tests - they should still pass
5. Repeat until the service just orchestrates

Each migration reveals how much of your "business logic" was actually just data access and transformation. The rules guide this process, helping both you and AI assistants generate models that encode reality rather than just structure data.

The end state? Your models become executable specifications. Reading them teaches your domain. Testing them verifies your business rules. And your services become so simple they barely need tests.

## Resources

- [Pydantic Documentation](https://docs.pydantic.dev/) - The foundation
- [Cursor Documentation](https://cursor.com/docs) - How rules work
