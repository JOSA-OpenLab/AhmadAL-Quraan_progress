# AAA testing pattern

* Arrange: Set up test data and condition
* Act: Execute the code 
* Assert: Verify the result

> The examples are mostly given in unit test, but this pattern is applied to any "set up → do something → check result" flow. Including integration test and E2E test.

Ex: 
```python
def test_parser_handles_trailing_comma():
# Arrange
parser = JSON5Parser()
input_text = '{"a": 1,}'
# Act
result = parser.parse(input_text)
# Assert
assert result == {"a": 1}
```

## **Why it matters:**

1. **Readability** — Anyone (including future you) can scan a test and instantly know what's setup vs. what's being tested vs. what's being checked, without reading line-by-line logic.
2. **One clear "Act" step keeps tests focused.** If you find yourself with multiple "Act" steps or actions scattered between assertions, it's often a sign the test is doing too much and should be split into separate tests.
3. **Debugging is faster.** When a test fails, you immediately know it failed at the "Assert" stage (expected vs actual mismatch) rather than the "Arrange" stage (setup broke) — these usually show up as different kinds of errors anyway, but the mental model helps you scan quickly.
4. **It maps naturally onto Hypothesis/pytest too** — e.g. with `@given`, the generated parameters _are_ the Arrange step, your function call is Act, and your property check is Assert.

**A common mistake to avoid:** cramming multiple "Act + Assert" pairs into a single test function to save time. It's tempting, but it usually means the test is actually testing multiple behaviors, and when it fails, you won't immediately know which behavior broke without reading closely. Generally: one behavior, one test, one clean AAA block.

Some people even add the `# Arrange` / `# Act` / `# Assert` comments literally (like above) to make it explicit — especially useful when you're learning, since it forces the structure. As you get more experienced, you'll often drop the comments because the structure becomes visually obvious from spacing/grouping alone, but the underlying discipline stays the same.
## Important rules
 
1. One assertion (or one logical assertion) per test. Multiple assertions hide which one
failed.
2. Names describe the behavior, not the function. test_parse is bad;
test_parser_handles_trailing_comma is good.
3. Tests are deterministic. No random data unless you use a seeded generator. No real
time (now()) — inject a clock. No real network — inject a transport
4. Each file should have  a separate **test file** for it.
