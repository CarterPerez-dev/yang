# Contributing to yang

Bug reports and pull requests are welcome at
https://github.com/CarterPerez-dev/yang.

A good bug report has the smallest input that shows the problem, the style
(or `.style.yang`) used, the output you got and the output you expected.

Pull requests should:

- keep yang's output on code that does not hit the bug byte-identical;
- add a test, usually a golden case in `yangtests/testdata/modern_syntax` or
  a test next to the existing ones for the same module;
- pass `python -m pytest yangtests` on the oldest and newest supported
  Python versions.

yang is licensed under the Apache License, Version 2.0. By contributing you
agree that your contribution is licensed under the same terms.

Fixes that also apply to YAPF are worth sending upstream to
https://github.com/google/yapf as well.
