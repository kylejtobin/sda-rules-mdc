# 🧠 Semantic Domain Architecture

A pattern so obvious you'll wonder why we ever did it differently.

[000-master-architecture.mdc](000-master-architecture.mdc) ← The complete pattern

If you've ever asked *"Why is all the business logic smeared across ten services?"*—here's your answer.

> **Semantic Domain Architecture** recognizes a simple truth: when your models truly encode reality, everything else—services, tests, infrastructure—becomes obvious mechanical work. The model IS the specification.

---

## 🤔 Want to Know More?

The full technical philosophy is published in  
[**The Semantic Layer We Never Knew We Were Building**](the_semantic_layer.md)

It reveals:
- How twenty years of "defensive programming" was actually reality encoding
- Why your constraints teach AI more than any prompt ever could
- How intelligent models make service layers nearly obsolete
- Why the best architecture emerges rather than being enforced

---

## 🔧 What You'll Find Here

A single `.mdc` file that changes how you think about architecture. Not rules to enforce, but a pattern so clear you'll see it everywhere once you know what to look for.

The core insight: Make your models so intelligent that AI (and developers) can read them and understand your entire domain. When `Temperature` knows thermodynamics and `Order` knows your business rules, architecture stops being something you impose and starts being something that emerges.

Use it with [Cursor](https://www.cursor.sh) for AI that understands your domain as well as you do—because your models teach it.

---

## 🧩 Why It Works

One pattern solves multiple problems:

- Business logic scattering → **Logic lives with data**
- Testing complexity → **Models test themselves**
- AI integration overhead → **Models already teach reality**
- Onboarding pain → **Read the models, understand everything**
- Service sprawl → **Services just orchestrate**

Your architecture becomes **self-evident, semantic, and impossible to get wrong**.

---

## 🧱 The Pattern

Models encode reality. Everything else follows.

```python
class Temperature(BaseModel):
    celsius: Decimal = Field(ge=-273.15)  # Absolute zero exists
    
    @computed_field
    @property
    def fahrenheit(self) -> Decimal:
        return self.celsius * Decimal('1.8') + 32
    
    def __add__(self, other: "Temperature") -> "Temperature":
        raise TypeError("Cannot add temperatures")  # That's not how physics works
```

This model teaches:
- Physical constraints (absolute zero)
- Unit conversions (celsius ↔ fahrenheit)
- Invalid operations (temperatures don't sum)

When AI reads this, it learns thermodynamics. When it reads your `Order` model, it learns your business. Services become trivial orchestration of intelligent models.

---

## 🚀 Getting Started

1. Create a `.cursor/` directory in your project root
2. Copy `000-master-architecture.mdc` into `.cursor/rules/000-master-architecture.mdc`
3. Watch as your models become teachers and your architecture becomes inevitable

You don't need to refactor everything at once. Start with one model. Make it truly intelligent. The path forward will become obvious.

---

## 🛠 Maintainer

Kyle ([@kylejtobin](https://github.com/kylejtobin)) – CTO, semantic application architect, sysadmin.  
Former barbecue overlord. Occasional woodworker.

---

## 📜 License

MIT, but you can't own patterns. They're the chords, not the music.