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

Nothing is provided. Translation runs on the model **you** choose — any provider your app lists in `app.Llm`
([`Osysharp.Llm`](../llm)) — with **your** key, which you give your app as a secret. Until the key has a value, `IsReady()` is
false: detection still works offline, nothing is sent anywhere, and every caller keeps the original text.

## What it does

| | |
|---|---|
| **Detection, offline** | `LanguageDetector` tells a text by its script — Arabic, Persian, Cyrillic, Greek, Hebrew, Devanagari, Thai, Japanese, Korean, Chinese — or by the commonest words and letters of twelve European languages. Free, instant and the same on every machine. It says when it is not sure |
| **The model only when needed** | `ModelTranslator.Detect` asks the model only when a text is too short or too mixed to tell, and uses its answer only when it is a language tag |
| **Translation through your model** | one plain call through `Llm.Complete` — your key, your router, your budgets, on your meter. No tools, no memory: the text is treated as text, never as instructions |
| **Terms kept** | names and product terms stay as written, or are written the way you say |
| **Long texts** | sent a paragraph at a time past `MaxChunk` |
| **Same language, no call** | "en-GB" into "en" is the text itself |
| **Not ready is a state** | with no key, `IsReady()` is false and `NotReadyReason()` says which secret to set — a caller keeps the original instead of failing |
| **Your languages** | add a `LanguageProfile` and the detector knows one more |

## Testing with it

No test should call a real model. Answer for the call with `CompleteWith.Stub((request, call) => new LlmResult { Response = Scripted.Text("…") })`, and make the translator
ready with `ModelKeyIsSet.Stub(name => true)`.

## Its tests

Detection across eighteen languages, the not-sure cases, a language an app adds, the prompt a translation sends and the
calls it never makes — every one with the model stubbed.

## Examples

Taken from this package's own tests, which compile and run on every release.

```osy
using Osysharp.MachineTranslation;

[Test]
void european_languages_are_told_by_their_words() {
  Assert.Equal("en", G("Hello, I have not received my booking and I would like to know where it is.").Language);
  Assert.Equal("sv", G("Hej, jag har inte fått min bokning och vill veta var den är.").Language);
  Assert.Equal("de", G("Hallo, ich habe meine Buchung nicht bekommen und möchte wissen, wo sie ist.").Language);
  Assert.Equal("fr", G("Bonjour, je n'ai pas reçu ma réservation et je voudrais savoir où elle est.").Language);
  Assert.Equal("es", G("Hola, no he recibido mi reserva y quiero saber dónde está.").Language);
  Assert.Equal("pt", G("Olá, não recebi a minha reserva e quero saber onde ela está.").Language);
  Assert.Equal("nl", G("Hallo, ik heb mijn boeking niet ontvangen en wil graag weten waar die is.").Language);
  Assert.Equal("pl", G("Cześć, nie dostałem mojej rezerwacji i chcę wiedzieć, gdzie jest.").Language);
  Assert.True(G("Hej, jag har inte fått min bokning och vill veta var den är.").Confident);
}
```

From [`tests/detector.test.osy`](tests/detector.test.osy).

```osy
using Osysharp.MachineTranslation;
using Osysharp.Llm;

[Test(ModelAnswers)]
void a_comment_is_translated_through_the_apps_model() {
  var t = new ModelTranslator();
  Assert.Equal("[en] Hotellet låg nära stranden.", t.Translate("Hotellet låg nära stranden.", "sv", "en", []));
  Assert.Equal(1, Asked.Count());
}
```

From [`tests/translator.test.osy`](tests/translator.test.osy).

```osy
using Osysharp.MachineTranslation;

[Test]
void a_swedish_message_full_of_letters_finnish_shares_is_sure() {
  var g = G("Hej! Jag försöker ladda ner fakturan för september men PDF:en öppnas aldrig. Kan ni skicka den igen?");
  Assert.Equal("sv", g.Language);
  Assert.True(g.Confident);
}
```

From [`tests/detector.test.osy`](tests/detector.test.osy).

## Learn more

- [The Osysharp.MachineTranslation reference](https://osysharp.com/reference/ui/machine-translation-kit/)
- [Osysharp.MachineTranslation in the package catalogue](https://osysharp.com/packages/#osysharp-machinetranslation)
- [The Osy# reference](https://osysharp.com/reference/) — the language, the runtime and every kit
- [`llms-full.txt`](https://osysharp.com/llms-full.txt) — the whole reference and specification as one Markdown file, for a coding agent

## Licence

MIT — see [`LICENSE`](LICENSE).
