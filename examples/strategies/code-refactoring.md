# Strategy: Code Refactoring

## When to Use

Refactoring a large function/module into smaller, testable units while ensuring existing tests still pass. Common in legacy code modernization, monolith decomposition, or pre-feature cleanup.

## Typical Policy

```yaml
max_retry: 3
max_replan: 2
phase_timeout: 20m
max_total_runtime: 60m
```

## Strategy Template

```yaml
goal: "Refactor {target_function_or_module} into N smaller units. All existing tests must pass. No behavioral changes."

constraints:
  - id: tests_pass
    type: script
    condition: "all existing test cases pass after refactor"
    verify: exec: cd {repo_path} && {test_command}
  - id: no_behavior_change
    type: semantic
    condition: "refactored code preserves original behavior exactly"
    verify: manual
  - id: function_size
    type: metric
    condition: "each new function ≤50 lines"
    verify: exec: python -c "
import ast, sys
with open('artifacts/refactored.py') as f:
    tree = ast.parse(f.read())
for node in ast.walk(tree):
    if isinstance(node, (ast.FunctionDef, ast.AsyncFunctionDef)):
        end = node.end_lineno or 0
        start = node.lineno
        if end - start > 50:
            print(f'{node.name}: {end-start} lines (>50)')
            sys.exit(1)
print('OK')
"

phases:
  - name: analyze
    actions: "Read the target file. Identify logical blocks, dependencies, and test coverage. Note any tight coupling that complicates extraction."
    consume: []
    output: [analysis.md]
    constraints: []

  - name: plan_refactor
    actions: "Based on analysis.md, design the refactored structure: which functions/classes to extract, their signatures, and how they compose. Write a refactoring plan with before/after structure."
    consume: [analysis.md]
    output: [refactor_plan.md]
    constraints:
      - id: plan_complete
        type: semantic
        condition: "plan covers all logical blocks identified in analysis, no gaps"
        verify: manual

  - name: implement
    actions: "Execute the refactoring plan. Write refactored code to artifacts/refactored.py. Update imports. Keep original behavior."
    consume: [analysis.md, refactor_plan.md]
    output: [refactored.py]
    constraints: [tests_pass, no_behavior_change, function_size]

  - name: verify
    actions: "Run full test suite. Compare output of original vs refactored on representative inputs. Write verification summary."
    consume: [refactored.py]
    output: [verification.md]
    constraints:
      - id: all_verified
        type: semantic
        condition: "verification summary confirms: tests pass, no behavior change, function size within limits"
        verify: manual
```

## Notes

- If tests are sparse, add a `write_tests` phase before `implement` to establish baseline coverage
- Common replan: original function too tightly coupled → replan to do partial extraction in stages
- For very large codebases, consider running multiple refactoring runs in parallel on different modules
