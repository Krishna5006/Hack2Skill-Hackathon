# Code Formatting Guide for Chat Messages

## How to Get Formatted Code in Chat Messages

Your chatbot now supports **formatted code blocks** similar to the structure shown in your image! Here's how it works:

## 1. Message Format Structure

To display code with proper formatting, your AWS Bedrock agent should return messages using **markdown code blocks**:

### Basic Format:
```
Here are 10 loop code examples in Java:

1. Simple for loop:
```java
for (int i = 0; i < 10; i++) {
    System.out.println(i);
}
```

2. For-each loop:
```java
int[] numbers = {1, 2, 3, 4, 5};
for (int number : numbers) {
    System.out.println(number);
}
```

3. While loop:
```java
int i = 0;
while (i < 10) {
    System.out.println(i);
    i++;
}
```
```

## 2. How It Works

The chatbot automatically:
- **Detects code blocks** wrapped in triple backticks (\`\`\`)
- **Extracts the language** (e.g., `java`, `python`, `javascript`)
- **Renders with proper formatting**:
  - Language header
  - Monospace font
  - Preserved whitespace and indentation
  - Syntax highlighting background
  - Scrollable for long code

## 3. Example Response from AWS Bedrock

Your AWS Bedrock agent should format responses like this:

```
Here are 10 loop code examples in Java:

1. Simple for loop:
```java
for (int i = 0; i < 10; i++) {
    System.out.println(i);
}
```

2. For-each loop:
```java
int[] numbers = {1, 2, 3, 4, 5};
for (int number : numbers) {
    System.out.println(number);
}
```

3. While loop:
```java
int i = 0;
while (i < 10) {
    System.out.println(i);
    i++;
}
```

4. Do-while loop:
```java
int i = 0;
do {
    System.out.println(i);
    i++;
} while (i < 10);
```

5. Nested for loop:
```java
for (int i = 0; i < 5; i++) {
    for (int j = 0; j < 5; j++) {
        System.out.print("* ");
    }
    System.out.println();
}
```

6. Infinite loop (using for):
```java
for (;;) {
    System.out.println("Infinite loop");
}
```

7. Infinite loop (using while):
```java
int i = 0;
while (true) {
    System.out.println(i);
    i++;
}
```

8. Loop with break:
```java
for (int i = 0; i < 10; i++) {
    if (i == 5) {
        break;
    }
    System.out.println(i);
}
```

9. Loop with continue:
```java
for (int i = 0; i < 10; i++) {
    if (i % 2 == 0) {
        continue;
    }
    System.out.println(i);
}
```

10. Loop with labeled break:
```java
outer: for (int i = 0; i < 5; i++) {
    for (int j = 0; j < 5; j++) {
        if (i * j > 6) {
            break outer;
        }
        System.out.println(i + " * " + j + " = " + (i * j));
    }
}
```
```

## 4. Testing Your Chatbot

To test the formatted code display:

1. **Open your browser** to `http://localhost:5173` (or your Vite dev server URL)
2. **Type in the chat**: "Show me loop examples in Java"
3. **Your AWS Bedrock agent** should respond with formatted code blocks
4. **The chat will display** each code block with:
   - A header showing the language (JAVA)
   - Properly formatted code with preserved indentation
   - Monospace font for readability

## 5. Supported Languages

You can use any language identifier in the code blocks:
- `java`
- `javascript`
- `python`
- `cpp`
- `c`
- `go`
- `rust`
- `typescript`
- etc.

## 6. Configuring AWS Bedrock Agent

To make your AWS Bedrock agent return properly formatted responses:

### Option 1: Update Agent Instructions
Add this to your agent's instructions:
```
When providing code examples, always format them using markdown code blocks.
Use triple backticks with the language name, like this:

```java
// Your code here
```

This ensures proper formatting in the user interface.
```

### Option 2: Use Knowledge Base
If using a knowledge base, ensure your documents include code examples formatted with markdown code blocks.

## 7. Current Implementation

The frontend now includes:

✅ **Code block detection** - Automatically finds code blocks in messages
✅ **Language extraction** - Identifies the programming language
✅ **Formatted rendering** - Displays code with proper styling
✅ **Line break preservation** - Maintains formatting for regular text
✅ **Responsive design** - Works on all screen sizes

## 8. Example Chat Flow

**User**: "Show me a for loop in Python"

**Bot Response** (formatted):
```
Here's a simple for loop in Python:

```python
for i in range(10):
    print(i)
```

This loop will print numbers from 0 to 9.
```

The chat will render this with:
- Regular text: "Here's a simple for loop in Python:"
- Code block with "PYTHON" header
- Formatted code with proper indentation
- Regular text: "This loop will print numbers from 0 to 9."

## 9. Troubleshooting

If code blocks aren't displaying properly:

1. **Check the message format** - Ensure triple backticks are used
2. **Verify language identifier** - Make sure it's on the same line as opening backticks
3. **Check for extra spaces** - No spaces between backticks and language name
4. **Test with simple example** - Try a basic code block first

## 10. Next Steps

To get the exact structure from your image:

1. **Configure your AWS Bedrock agent** to return responses with markdown code blocks
2. **Test with various prompts** like:
   - "Show me loop examples in Java"
   - "Give me Python function examples"
   - "Explain recursion with code"
3. **Adjust styling** in `App.css` if needed for your preferred look

---

Your chatbot is now ready to display beautifully formatted code examples just like in your reference image! 🎉
