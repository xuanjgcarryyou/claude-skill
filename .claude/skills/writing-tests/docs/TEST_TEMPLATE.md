# Test Templates by Framework

Reference templates for the `writing-tests` skill. Use the section matching your project's test runner.

---

## Jest / Vitest (TypeScript / JavaScript)

```typescript
// tests/unit/myModule.test.ts
import { functionName } from '../src/myModule';

describe('functionName', () => {
  // ── Happy Path ──────────────────────────────────────────────
  it('should return expected value when given valid input', () => {
    const result = functionName(validInput);
    expect(result).toEqual(expectedOutput);
  });

  // ── Boundary Cases ───────────────────────────────────────────
  it('should return null when input is null', () => {
    expect(functionName(null)).toBeNull();
  });

  it('should return empty array when input is empty array', () => {
    expect(functionName([])).toEqual([]);
  });

  it('should handle zero correctly', () => {
    expect(functionName(0)).toEqual(/* expected */);
  });

  it('should handle maximum length string without truncation', () => {
    const maxInput = 'a'.repeat(MAX_LENGTH);
    expect(() => functionName(maxInput)).not.toThrow();
  });

  // ── Error Paths ───────────────────────────────────────────────
  it('should throw InvalidInputError when input type is wrong', () => {
    expect(() => functionName(123 as any)).toThrow(InvalidInputError);
  });

  it('should return 503 when downstream service is unavailable', () => {
    mockService.mockRejectedValueOnce(new Error('Connection refused'));
    await expect(functionName(validInput)).rejects.toThrow('Connection refused');
  });
});
```

**File location:** `tests/unit/<moduleName>.test.ts` or `src/<moduleName>.test.ts` (co-located)

---

## pytest (Python)

```python
# tests/unit/test_my_module.py
import pytest
from src.my_module import function_name, InvalidInputError

class TestFunctionName:
    # ── Happy Path ──────────────────────────────────────────────
    def test_returns_expected_value_when_given_valid_input(self):
        result = function_name(valid_input)
        assert result == expected_output

    # ── Boundary Cases ───────────────────────────────────────────
    def test_returns_none_when_input_is_none(self):
        assert function_name(None) is None

    def test_handles_empty_string(self):
        assert function_name("") == expected_for_empty

    def test_handles_zero(self):
        assert function_name(0) == expected_for_zero

    def test_handles_maximum_length_input(self):
        max_input = "a" * MAX_LENGTH
        result = function_name(max_input)
        assert len(result) <= MAX_LENGTH

    # ── Error Paths ───────────────────────────────────────────────
    def test_raises_invalid_input_error_when_type_is_wrong(self):
        with pytest.raises(InvalidInputError):
            function_name(123)

    def test_raises_connection_error_when_service_unavailable(self, mock_service):
        mock_service.side_effect = ConnectionError("Service down")
        with pytest.raises(ConnectionError):
            function_name(valid_input)
```

**File location:** `tests/unit/test_<module_name>.py`

---

## Go (testing package)

```go
// internal/mypackage/mymodule_test.go
package mypackage_test

import (
    "testing"
    "github.com/stretchr/testify/assert"
    "yourproject/internal/mypackage"
)

// ── Happy Path ──────────────────────────────────────────────
func TestFunctionName_ReturnsExpectedValue_WhenGivenValidInput(t *testing.T) {
    result, err := mypackage.FunctionName(validInput)
    assert.NoError(t, err)
    assert.Equal(t, expectedOutput, result)
}

// ── Boundary Cases ───────────────────────────────────────────
func TestFunctionName_ReturnsError_WhenInputIsEmpty(t *testing.T) {
    _, err := mypackage.FunctionName("")
    assert.ErrorIs(t, err, mypackage.ErrEmptyInput)
}

func TestFunctionName_HandlesZero(t *testing.T) {
    result, err := mypackage.FunctionName(0)
    assert.NoError(t, err)
    assert.Equal(t, expectedForZero, result)
}

// ── Error Paths ───────────────────────────────────────────────
func TestFunctionName_ReturnsError_WhenInputIsNil(t *testing.T) {
    _, err := mypackage.FunctionName(nil)
    assert.ErrorIs(t, err, mypackage.ErrNilInput)
}
```

**File location:** same package directory, `*_test.go`

---

## RSpec (Ruby)

```ruby
# spec/lib/my_module_spec.rb
require 'rails_helper'

RSpec.describe MyModule::FunctionName do
  subject(:result) { described_class.call(input) }

  # ── Happy Path ──────────────────────────────────────────────
  context 'when given valid input' do
    let(:input) { valid_input }

    it 'returns the expected value' do
      expect(result).to eq(expected_output)
    end
  end

  # ── Boundary Cases ───────────────────────────────────────────
  context 'when input is nil' do
    let(:input) { nil }

    it 'returns nil without raising' do
      expect(result).to be_nil
    end
  end

  context 'when input is an empty string' do
    let(:input) { '' }

    it 'returns the empty-case value' do
      expect(result).to eq(expected_for_empty)
    end
  end

  # ── Error Paths ───────────────────────────────────────────────
  context 'when input type is invalid' do
    let(:input) { 123 }

    it 'raises InvalidInputError' do
      expect { result }.to raise_error(MyModule::InvalidInputError)
    end
  end
end
```

**File location:** `spec/lib/<module_name>_spec.rb`

---

## Boundary Case Reference Table

| Boundary | What to test |
|----------|-------------|
| `null` / `nil` / `None` | Input is null where a value is expected |
| Empty | `""`, `[]`, `{}` as input |
| Zero | Numeric `0` where a positive value is expected |
| Maximum length | String at max allowed length; one character over max |
| Minimum value | Smallest allowed numeric input |
| Whitespace-only | `"   "` or `"\t\n"` as input |
| Duplicate input | Same value submitted twice |
| Unicode / special chars | Input containing emoji (`😀`), accented chars (`café`) |

---

## Mocking Reference

| What | Jest/Vitest | pytest | Go | RSpec |
|------|------------|--------|-----|-------|
| External HTTP | `jest.fn()` / `vi.fn()` | `unittest.mock.patch` | `httptest.Server` | `WebMock` |
| System clock | `jest.useFakeTimers()` | `freezegun` | `clock.Now` override | `Timecop` |
| File system | `jest.mock('fs')` | `tmp_path` fixture | `afero` | `FakeFS` |
| DB (unit test) | Mock repository | `MagicMock` | Interface mock | `double` |
| DB (integration) | Real test DB | Real test DB | Real test DB | Real test DB |
