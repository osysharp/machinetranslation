# Osysharp.MachineTranslation

**Which language is this — and what does it say in mine?** Language detection that works offline, and translation
through your app's own model.

## Use it

```osy
app Guide {
  use Osysharp.MachineTranslation@0;
  model "model/**/*.osy";
}
```

```osy
using Osysharp.MachineTranslation;

var t = new ModelTranslator { KeySecret = "ModelKey" };
var language = t.Detect("Hej! Jag har inte fått min bokning.");               // "sv" — no call
var english  = t.Translate(text, language, "en", [new TranslationTerm { Term = "Wanderly" }]);
```

## Your provider, your key

Nothing is provided. Translation runs on the model **you** choose — any provider your app allows (`app.Models`,
`app.DefaultModel`) — with **your** key, which you give your app as a secret. Until the key has a value, `IsReady()` is
false: detection still works offline, nothing is sent anywhere, and every caller keeps the original text.

## What it does

| | |
|---|---|
| **Detection, offline** | `LanguageDetector` tells a text by its script — Arabic, Persian, Cyrillic, Greek, Hebrew, Devanagari, Thai, Japanese, Korean, Chinese — or by the commonest words and letters of twelve European languages. Free, instant and the same on every machine. It says when it is not sure |
| **The model only when needed** | `ModelTranslator.Detect` asks the model only when a text is too short or too mixed to tell, and uses its answer only when it is a language tag |
| **Translation through your model** | one plain call through `LlmClient.Complete` — your key, your router, your budget. No tools, no memory: the text is treated as text, never as instructions |
| **Terms kept** | names and product terms stay as written, or are written the way you say |
| **Long texts** | sent a paragraph at a time past `MaxChunk` |
| **Same language, no call** | "en-GB" into "en" is the text itself |
| **Not ready is a state** | with no key, `IsReady()` is false and `NotReadyReason()` says which secret to set — a caller keeps the original instead of failing |
| **Your languages** | add a `LanguageProfile` and the detector knows one more |

## Testing with it

No test should call a real model. Answer for it with `LlmClient.Complete.Stub(prompt => …)`, and make the translator
ready with `ModelKeyIsSet.Stub(name => true)`.

## Its tests

Detection across eighteen languages, the not-sure cases, a language an app adds, the prompt a translation sends and the
calls it never makes — every one with the model stubbed.
