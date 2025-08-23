# 🧠 Semantic Domain Architecture

[![Cursor Compatible](https://img.shields.io/badge/Cursor-Compatible-00A6FF.svg)](https://cursor.com)
[![Rules: SDA](https://img.shields.io/badge/Rules-SDA-purple.svg)](https://github.com/kylejobin/sda-rules-mdc)
[![MIT License](https://img.shields.io/github/license/kylejtobin/sda-rules-mdc)](LICENSE)

Python domain modeling that doesn't suck. Cursor rules included.

## Quick Start

```bash
git clone https://github.com/kylejobin/sda-rules-mdc.git
cp -r sda-rules-mdc/.cursor .
```

Now Cursor understands that your models should actually model things.

## Why This Exists

Picture this: It's 2am. Production is on fire. You're tracing through a bug that makes no sense. The stack trace leads you through `OrderService`, then `OrderValidator`, then `OrderHelper`, then `order_utils.py`, and finally you find it—the actual business logic, scattered across seven files like someone threw it in a blender.

The bug? Someone passed a price as a string instead of a float. But really, the bug was that your Order model doesn't know what an order is. It's just a property bag, a glorified dictionary with type hints. The actual intelligence—what makes an order an *order*—lives everywhere except where it should.

We've been writing code backwards. We put data in models and logic in services because someone told us about "separation of concerns" in 2003 and we never questioned it. But here's the thing: **the biggest concern is that your domain logic has no home**.

Remember when "enterprise architecture" meant adding seventeen layers of abstraction to hide a database query? When every simple operation required a Factory that created a Manager that used a Service that called a Repository that wrapped a DAO? We traded simplicity for ceremony and called it "best practices."

Well, I'm done pretending that's sane.

## What You Get

Six focused rules that teach Cursor (and you) how to write models that model:

### Core Rules (Always Active)
- **[`000-sda-core.mdc`](.cursor/rules/000-sda-core.mdc)** - The complete philosophy. Domain modeling identity, constructive proof, complexity budget, and zero tolerance policies.
- **[`010-type-dispatch.mdc`](.cursor/rules/010-type-dispatch.mdc)** - Discriminated unions and smart enums with concrete code examples. The pattern that eliminates conditionals.

### Implementation Rules (Context-Aware)
- **[`020-domain-modeling.mdc`](.cursor/rules/020-domain-modeling.mdc)** - Rich types with Pydantic. Computed fields, immutable transitions, intelligent models.
- **[`030-protocol-services-boundary.mdc`](.cursor/rules/030-protocol-services-boundary.mdc)** - Complete PSB module architecture. Ownership-first boundaries, service rules, validation checklists.
- **[`040-testing.mdc`](.cursor/rules/040-testing.mdc)** - Test domain intelligence, not plumbing. Focus on decisions, not tools.
- **[`050-python-standards.mdc`](.cursor/rules/050-python-standards.mdc)** - Type safety with zero tolerance. Modern Python 3.13+. Tooling enforcement.

### Why This Structure?

Cursor's context window isn't infinite. Loading every rule for every file is like bringing an entire library to look up one fact. This consolidated design means:
- **Core philosophy** always active - domain modeling identity and zero tolerance policies
- **Type dispatch patterns** with concrete examples when working with conditionals and enums
- **Context-aware rules** load only when relevant to your current work
- **Maximum precision** with minimal cognitive overhead
- More room for your actual code in the context

It's the difference between a focused assistant and one that's read too much and can't shut up about it.

## The Core Philosophy: Software by Subtraction

Your models should be domain experts, not data buckets. But here's the real secret: SDA works not because of what it adds, but because of what it refuses to do. Every pattern you don't need. Every layer you don't add. Every abstraction you don't create.

```python
# What we write (data bucket)
class Temperature:
    celsius: float
    
# What we should write (domain expert)
class Temperature(BaseModel):
    celsius: Decimal = Field(ge=-273.15)  # Knows absolute zero
    
    @property
    def fahrenheit(self) -> Decimal:
        return self.celsius * Decimal('1.8') + 32
    
    @property
    def state_of_water(self) -> WaterState:
        if self.celsius <= 0:
            return WaterState.SOLID
        elif self.celsius >= 100:
            return WaterState.GAS
        return WaterState.LIQUID
```

One teaches you about temperature. The other one just... holds a number.

## The Patterns That Matter

### 1. Rich Types Over Primitive Obsession

Every time you type `amount: float`, somewhere a developer cries debugging a currency mismatch. I've seen production systems lose thousands of dollars because someone forgot that the payment gateway expects cents but the invoice system uses dollars. True story.

```python
# The bug factory
def process_payment(amount: float, currency: str) -> float:
    # Hope everyone remembers amount is in cents!
    # Hope everyone passes currency in the same format!
    # Hope no one does math without considering currency!
    ...

# The domain model
class Money(BaseModel):
    amount: Decimal
    currency: Currency
    
    def __add__(self, other: Money) -> Money:
        if self.currency != other.currency:
            raise ValueError(f"Cannot add {self.currency} to {other.currency}")
        return Money(amount=self.amount + other.amount, currency=self.currency)
    
    def split_evenly(self, ways: int) -> list[Money]:
        """No lost pennies in rounding."""
        base_amount = (self.amount / ways).quantize(Decimal('0.01'))
        remainder = self.amount - (base_amount * ways)
        
        splits = [Money(base_amount, self.currency) for _ in range(ways)]
        # Add the remainder pennies to the first split
        if remainder:
            splits[0] = Money(base_amount + remainder, self.currency)
        return splits
```

Now `money + money` either works correctly or fails loudly. No silent bugs. No "wait, was that in cents or dollars?" at 3am.

### 2. Behavior Lives With Data

Stop writing this:

```python
class Order:
    items: list[dict]
    status: str

class OrderService:
    def calculate_total(self, order: Order) -> float:
        # 50 lines of logic
    
    def can_cancel(self, order: Order) -> bool:
        # 30 lines of logic
    
    def apply_discount(self, order: Order, discount: float) -> float:
        # 40 lines of logic
```

Write this:

```python
class Order(BaseModel):
    items: list[LineItem]
    status: OrderStatus
    
    @computed_field
    @property
    def total(self) -> Money:
        return sum((item.total for item in self.items), Money.zero())
    
    def cancel(self) -> "Order":
        return self.status.cancel_order(self)
```

The order knows how to be an order. Revolutionary concept, I know.

You know what's wild? Half the developers I show this to act like I just invented fire. "You can put methods... on your models?" Yes. Yes you can. It's been possible since roughly 1967 when Ole-Johan Dahl and Kristen Nygaard invented object-oriented programming. We just... forgot.

### 3. Constructive Type Transformation (The Heart of SDA)

Every type transition must be provable through construction, not assertion. This is the difference between hoping your code works and knowing it does.

```python
# The assertion approach (hope)
def process_payment(data: dict) -> Payment:
    # "Trust me, this dict has the right shape"
    return cast(Payment, data)  # type: ignore

# The constructive approach (proof)
def process_payment(data: dict) -> Payment:
    # Step 1: Unpack explicitly
    amount_raw = data.get("amount")
    currency_raw = data.get("currency")
    
    # Step 2: Transform to domain types
    amount = Money.from_string(amount_raw) if amount_raw else Money.zero()
    currency = Currency(currency_raw) if currency_raw else Currency.USD
    
    # Step 3: Construct - if this succeeds, it's correct
    return Payment(amount=amount, currency=currency)
```

No casts. No type ignores. No "trust me bro" comments. The successful construction IS the proof.

### 4. Make Invalid States Unrepresentable (Discriminated Unions)

The pattern that eliminates entire categories of bugs. Instead of defensive programming, use types that make bugs impossible.

Remember every time you've written code like this?

```python
# The defensive programming nightmare
def send_notification(user_data: dict) -> None:
    # First, let's check if we have what we need...
    if "email" in user_data:
        # But wait, is it actually an email?
        if "@" in user_data["email"]:  # Top-tier email validation right here
            # Do we have a subject?
            subject = user_data.get("email_subject", "No Subject")
            # What about the body?
            body = user_data.get("email_body", "")
            send_email(user_data["email"], subject, body)
    elif "phone" in user_data:
        # Is it a valid phone number? Who knows!
        send_sms(user_data["phone"], user_data.get("sms_message", ""))
    else:
        # The famous "this should never happen" comment
        raise ValueError("No contact method")  # Narrator: It happened.
```

versus:

```python
# The SDA way: discriminated unions
class EmailNotification(BaseModel):
    type: Literal["email"] = "email"
    address: EmailAddress
    subject: str = Field(min_length=1)
    body: str
    
    def send(self) -> NotificationResult:
        # Can't forget the subject - it's required!
        # Can't pass an invalid email - EmailAddress validates!
        # Can't mix up SMS and email - they're different types!
        return send_email(self.address, self.subject, self.body)

class SmsNotification(BaseModel):
    type: Literal["sms"] = "sms"
    phone: PhoneNumber
    message: str = Field(max_length=160)  # Remember SMS limits?
    
    def send(self) -> NotificationResult:
        return send_sms(self.phone, self.message)

Notification = Annotated[
    Union[EmailNotification, SmsNotification],
    Field(discriminator="type")
]

# Now this is impossible to mess up
def send_notification(notification: Notification) -> NotificationResult:
    return notification.send()
```

No more "what if the dict doesn't have the key I expect?" Just types that work.

The magic is in that `discriminator="type"`. Pydantic looks at the type field and automatically creates the right class. No isinstance checks, no manual dispatch. The type system is your router.

### 5. State Machines That Actually State Machine

Your business logic is full of state machines. Order workflows, user lifecycles, payment flows. But I bet they're implemented as string comparisons scattered across service methods, held together by hopes and comments that say "TODO: refactor this."

```python
class PaymentStatus(StrEnum):
    PENDING = "pending"
    PROCESSING = "processing"
    COMPLETED = "completed"
    FAILED = "failed"
    REFUNDED = "refunded"
    
    def can_refund(self) -> bool:
        return self == self.COMPLETED
    
    def can_retry(self) -> bool:
        return self == self.FAILED
    
    def next_status(self, event: PaymentEvent) -> "PaymentStatus":
        transitions = {
            (self.PENDING, PaymentEvent.PROCESS): self.PROCESSING,
            (self.PROCESSING, PaymentEvent.SUCCEED): self.COMPLETED,
            (self.PROCESSING, PaymentEvent.FAIL): self.FAILED,
            (self.COMPLETED, PaymentEvent.REFUND): self.REFUNDED,
            (self.FAILED, PaymentEvent.RETRY): self.PENDING,
        }
        
        next_status = transitions.get((self, event))
        if next_status is None:
            raise InvalidTransition(
                f"Cannot {event} from {self}. "
                f"Valid events: {self._valid_events()}"
            )
        return next_status
    
    def _valid_events(self) -> set[PaymentEvent]:
        # Now you can actually ask "what can I do from here?"
        return {
            event for (status, event), _ in self._transitions.items() 
            if status == self
        }
```

Your enum now encodes your entire payment flow. One source of truth. No more "wait, can a refunded payment be processed again?" debates at sprint planning.

## We've All Written This Code

Let's fix the code we've all written at some point:

```python
# The "architecture" we've all built
class Product:
    sku: str
    name: str
    price: float
    discount_percentage: float
    stock_level: int
    reserved_stock: int
    category: str
    
class ProductService:
    def get_available_stock(self, product: Product) -> int:
        return product.stock_level - product.reserved_stock
    
    def calculate_price(self, product: Product) -> float:
        if product.discount_percentage > 0:
            return product.price * (1 - product.discount_percentage / 100)
        return product.price
    
    def can_purchase(self, product: Product, quantity: int) -> tuple[bool, str]:
        if quantity <= 0:
            return False, "Invalid quantity"
        
        available = self.get_available_stock(product)
        if available < quantity:
            return False, f"Only {available} items available"
            
        if product.category == "limited_edition" and quantity > 2:
            return False, "Limited edition items have a maximum quantity of 2"
            
        return True, "OK"
    
    def reserve_stock(self, product: Product, quantity: int) -> Product:
        # Mutates the product! What could go wrong?
        product.reserved_stock += quantity
        return product
```

And here's what it becomes with SDA:

```python
class StockLevel(BaseModel):
    total: Count = Field(ge=0)
    reserved: Count = Field(ge=0)
    
    @field_validator('reserved')
    @classmethod
    def reserved_cannot_exceed_total(cls, v: Count, info: ValidationInfo) -> Count:
        if total := info.data.get('total'):
            if v > total:
                raise ValueError('Reserved cannot exceed total stock')
        return v
    
    @computed_field
    @property
    def available(self) -> Count:
        return Count(self.total - self.reserved)
    
    def reserve(self, quantity: Count) -> "StockLevel":
        """Immutable reservation - returns new StockLevel"""
        if quantity > self.available:
            raise InsufficientStock(requested=quantity, available=self.available)
        return self.model_copy(update={
            'reserved': self.reserved + quantity
        })

class ProductCategory(StrEnum):
    STANDARD = "standard"
    LIMITED_EDITION = "limited_edition"
    CLEARANCE = "clearance"
    
    @property
    def max_purchase_quantity(self) -> Count | None:
        """Some categories have purchase limits"""
        limits = {
            self.LIMITED_EDITION: Count(2),
            self.STANDARD: None,
            self.CLEARANCE: None
        }
        return limits[self]

class Price(BaseModel):
    amount: Money
    discount: Percent = Percent(0)
    
    @computed_field
    @property
    def final_amount(self) -> Money:
        """The actual price after discount"""
        if self.discount == Percent(0):
            return self.amount
        discount_multiplier = (Percent(100) - self.discount) / Percent(100)
        return self.amount * discount_multiplier

class Product(BaseModel):
    sku: ProductSku
    name: str
    price: Price
    stock: StockLevel
    category: ProductCategory
    
    model_config = {"frozen": True}
    
    def check_purchase_eligibility(self, quantity: Count) -> PurchaseEligibility:
        """All purchase rules in one place"""
        if quantity <= 0:
            return PurchaseEligibility.invalid_quantity()
            
        if quantity > self.stock.available:
            return PurchaseEligibility.insufficient_stock(
                requested=quantity,
                available=self.stock.available
            )
            
        if max_qty := self.category.max_purchase_quantity:
            if quantity > max_qty:
                return PurchaseEligibility.exceeds_limit(max_qty)
                
        return PurchaseEligibility.eligible()
    
    def reserve_stock(self, quantity: Count) -> "Product":
        """Returns new Product with updated stock"""
        return self.model_copy(update={
            'stock': self.stock.reserve(quantity)
        })
```

What changed? Everything:
- No more float prices (was it dollars? cents? euros? bitcoin?)
- Stock logic lives with stock data
- Categories know their own rules
- Immutable operations (no more "who changed my product?!" debugging sessions)
- The model teaches you the business rules

## The Cultural Shift

Here's what they don't tell you about adopting SDA: it changes how your team talks about code.

Before SDA, standups sound like this: "I'm adding validation to the OrderService. Jim's working on the OrderValidationHelper. Sarah's refactoring the OrderProcessingManager." We're describing plumbing, not the problem.

After SDA: "I'm teaching Orders how to apply bulk discounts. Jim's implementing the subscription renewal state machine. Sarah's modeling the return merchandise workflow."

See the difference? We stop talking about services and start talking about the business. Product managers can actually understand our standups. Hell, they start *attending* them.

The junior developers get productive faster. Why? Because instead of navigating a maze of service classes, they can look at a model and understand the business rules. `PaymentStatus.can_refund()` doesn't require a PhD in Architecture Astronautics to understand.

But the best part? The arguments stop. Not completely (we're still developers), but the religious wars about "where should this logic live?" just... end. There's an obvious answer now: **with the data it operates on. The code itself enforces the architecture.**

One warning though: once your team tastes this, they can't go back. I've watched developers leave companies rather than return to anemic models. It's like asking someone who's discovered type safety to go back to stringly-typed everything. Some knowledge can't be unlearned.

## Dealing with the Messy Outside World

Here's the thing nobody tells you: the outside world doesn't follow SDA. Your database returns dictionaries. APIs send you JSON. Legacy systems speak XML from 2003.

The trick is boundary intelligence - convert external data to domain models at the edges:

```python
# Bad: Letting external structures infect your domain
def process_api_order(api_data: dict) -> None:
    if api_data.get("type") == "standard":
        # Now dict surgery spreads through your codebase
        process_standard_order(api_data)
    elif api_data.get("type") == "express":
        process_express_order(api_data)

# Good: Convert at the boundary
def process_api_order(api_data: dict) -> None:
    # One place to handle external messiness
    order = parse_external_order(api_data)  # Returns proper Order model
    # Now work with type-safe domain models
    result = order.process()
    
def parse_external_order(data: dict) -> Order:
    """The DMZ between external chaos and internal sanity."""
    # This is the ONLY place that knows about external format
    order_type = data.get("type", "standard")
    
    # Convert to domain types immediately
    return Order(
        type=OrderType(order_type),
        items=[parse_line_item(item) for item in data.get("items", [])],
        customer=parse_customer(data["customer"])
    )
```

Keep the mess at the borders. Your core domain stays clean.

## Testing (The F1 Car Analogy)

Your SDA models are like F1 cars - sophisticated machines with the complexity concentrated in specific areas. You don't test that the engine block is made of metal. You test that it produces the right power at the right RPM.

```python
# Don't test Pydantic's validation
def test_money_validates_positive():
    # Pydantic already tested this
    with pytest.raises(ValidationError):
        Money(amount=-1, currency=Currency.USD)

# Test your domain logic
def test_money_splits_evenly_with_remainder():
    money = Money(amount=Decimal("10.03"), currency=Currency.USD)
    splits = money.split_evenly(3)
    
    # First split gets the remainder penny
    assert splits[0].amount == Decimal("3.35")
    assert splits[1].amount == Decimal("3.34")
    assert splits[2].amount == Decimal("3.34")
    
    # No money lost in rounding
    assert sum(s.amount for s in splits) == money.amount
```

Test the intelligence, not the plumbing.

## The Mental Shift

The hardest part isn't learning Pydantic. It's unlearning the "anemic model" pattern that's been drilled into us for decades. 

Stop thinking in layers. Start thinking in concepts. An Order isn't a data structure that needs a service to tell it what to do. An Order is a domain concept that knows how to be an order.

Your services should shrink to almost nothing—just orchestration and infrastructure. If a service method is more than 10 lines, ask yourself: what domain concept is trying to emerge?

## Start Tomorrow

Here's your migration path that actually works:

1. **Pick one painful primitive**. You know the one. That `user_type: str` that's compared against magic strings in 47 places. The one where someone inevitably types "Admin" instead of "admin" and breaks production.

2. **Make it an enum with one method**. Just one. Maybe `is_admin()`. Nothing fancy.

3. **Find every place that string is compared**. Your IDE can help here. Move that logic to the enum.

4. **Watch your code get smaller** and your bugs disappear. 

5. **Show your team**. This is important. Nothing sells SDA like deleting 50 lines of defensive programming.

6. **Repeat** until your models actually model your domain.

The beauty is you can do this incrementally. You don't need permission to make a value object. You don't need a migration plan to add a method to a model. Just start.

## When to Use This (And When Not To)

**SDA shines when:**
- Your domain has actual business rules (not just CRUD)
- The same logic appears in multiple places
- You're debugging "state" bugs constantly
- New developers ask "but what are the rules?"
- Your tests are more mock than test

**SDA is overkill when:**
- You're literally just storing and retrieving data
- The "domain" is just "save this form to the database"
- You're prototyping and everything changes hourly
- The complexity is in the infrastructure, not the domain

Don't use a Formula 1 car to go to the grocery store. But also don't use a grocery getter on a race track.

## The Part Where I Get Real

Bad code isn't usually written by bad developers. It's written by good developers working in bad systems. Systems that force you to scatter logic across seventeen files. Systems that make you write unit tests that are 90% mocks. Systems that turn simple business rules into distributed computing problems.

SDA isn't about being clever. It's about being honest. Honest about what your code actually does. Honest about where complexity lives. Honest about the fact that `validate_order_status_transition_rules_for_premium_customers()` probably belongs on the Order model, not in a utility class.

When you adopt SDA, you're not just changing your code. You're changing how you think about code. You're admitting that maybe, just maybe, the objects in "object-oriented programming" should actually... orient around objects.

## Tools

- **[Pydantic V2](https://docs.pydantic.dev/)**: Non-negotiable. If you're on V1, migrate. The performance alone is worth it.
- **[Cursor](https://cursor.com/)**: These rules teach it to write SDA-style code. Like pair programming with someone who's read all the docs.
- **[mypy](http://mypy-lang.org/)**: `--strict` mode or go home. Type systems only help if you use them.
- **[Hypothesis](https://hypothesis.readthedocs.io/)**: Property testing + smart models = bugs found before QA does.

## Contributing

Found a pattern that makes models smarter? PR it. Got war stories about migrating from anemic models? Share them. This is a living architecture that gets better when people who've been in the trenches contribute.

---

*Your models should be so good that your business analysts steal them for documentation. If you have to explain what your model does, it's not doing enough.*