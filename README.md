# module-template-go

Botmother tashqi moduli — Go template.

Bu repo yangi modul yaratish uchun boshlang'ich nuqta. `botmodule-go` SDK'ni
ishlatadi va uchta node turi ko'rsatadi: Echo (action), AuthHeader (credential),
OnKeyword (trigger).

---

## Tez boshlash

```bash
git clone https://github.com/yourorg/module-template-go
cd module-template-go
```

**O'zgartirish kerak bo'lgan joylar:**

1. `module.yaml` — `module.id` (global unique slug), `name`, `author`, `github`.
2. `main.go` — `moduleID` konstanta va barcha `"mymodule.*"` qiymatlarni
   o'z modulId'ingizga almashtiring.
3. Node'larni o'zgartiring yoki yangisini qo'shing.

```bash
# Lokal test
go run .

# Boshqa terminaldagi test
curl http://localhost:8100/health
curl -X POST http://localhost:8100/rpc \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"describe","id":1}'
```

---

## Modul yaratish qadamlari

### 1. ID va nom o'zgartirish

`module.yaml`:
```yaml
module:
  id: mymodule        # o'zgartiring: global unique slug
  name: My Module     # ko'rinadigan nom
```

`main.go`:
```go
const moduleID = "mymodule"  // module.yaml bilan mos bo'lsin
```

### 2. Node qo'shish

```go
m.AddNode(botmodule.Node{
    Type:     "mymodule.MyNode",      // moduleID.NodeName format
    Title:    "Mening node'im",
    Category: "integrations",
    Icon:     "box",
    Color:    "blue",
    Content: []botmodule.Field{
        {Type: "text", Key: "param1", Label: "Parametr"},
    },
    Defaults: map[string]any{"param1": ""},
    Execute: func(c *botmodule.ExecuteCtx) botmodule.Result {
        val := c.String("param1")
        return botmodule.Result{
            ContextUpdates: map[string]any{"result": val},
        }
    },
})
```

### 3. Trigger qo'shish

```go
m.AddNode(botmodule.Node{
    Type:        "mymodule.OnEvent",
    Title:       "Hodisa kelganda",
    Category:    "triggers",
    Trigger:     true,
    TriggerMode: "event-match",
    Content: []botmodule.Field{
        {Type: "text", Key: "pattern", Label: "Pattern"},
    },
    Match: func(c *botmodule.TriggerCtx) botmodule.MatchResult {
        text := c.MessageText()
        pattern := c.String("pattern")
        return botmodule.MatchResult{
            Matched: strings.Contains(text, pattern),
        }
    },
})
```

### 4. Credential ishlatish

```go
// Manifest:
Field{Type: "credential", Key: "api_cred", Label: "API Credential"}

// Handler:
Execute: func(c *botmodule.ExecuteCtx) botmodule.Result {
    cred, ok := c.Credential("api_cred")
    if !ok {
        return botmodule.Result{ContextUpdates: map[string]any{}}
    }
    apiKey := cred.Data["api_key"]
    // ... HTTP so'rov yuboring
}
```

---

## Docker build

```bash
docker build -t registry.iprogrammer.uz/botspace/mymodule:latest .
docker push registry.iprogrammer.uz/botspace/mymodule:latest
```

`go.sum` yo'q bo'lsa avval generate qiling:
```bash
go mod tidy
```

---

## Prodga o'tish (lokal replace olib tashlash)

`go.mod` ichidagi `replace` qatorini olib tashlang:
```
// Bu qatorni o'chiring:
replace github.com/BotSpace/botmodule-go => ../botmodule-go
```
Keyin:
```bash
go mod tidy
```

---

## Admin panel orqali ro'yxatga olish

1. Image'ni registryga push qiling.
2. Admin panel → Modullar → Modul qo'shish:
   - `slug` = `module.id` (module.yaml)
   - `image` = registry yo'li (masalan `registry.iprogrammer.uz/botspace/mymodule:latest`)
   - `port` = 8100
   - `active` = true
3. Kerak bo'lsa `auth_token` kiriting — module pod `MODULE_AUTH_TOKEN` env sifatida oladi.

---

## SDK hujjati

To'liq API: [botmodule-go README](../botmodule-go/README.md) yoki
`github.com/BotSpace/botmodule-go`.
