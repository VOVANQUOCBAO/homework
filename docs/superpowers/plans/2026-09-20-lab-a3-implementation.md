# Lab A3 Android Application Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a submission-ready Android Java + XML project for MSSV 231A290036 with portrait, landscape, ConstraintLayout, dark-mode, reusable-style, and report deliverables.

**Architecture:** A single Android application module contains two activities and resource-qualified layouts. Python structural tests parse source/XML without an emulator; Gradle then performs Android resource linking, Java compilation, lint, and unit tests when an Android SDK is available.

**Tech Stack:** Java 17, Android Gradle Plugin 8.7.3, Gradle 8.9, compileSdk 35, minSdk 24, AndroidX AppCompat 1.7.0, ConstraintLayout 2.2.0, Material Components 1.12.0, Python 3 standard library XML parser.

**Spec:** `docs/superpowers/specs/2026-09-20-lab-a3-design.md`

## Global Constraints

- Package and namespace: `vn.edu.vhu.ltdd.a3layout`.
- Application name: `A3_231A290036`.
- Student name: `Võ Văn Quốc Bảo`.
- Student ID: `231A290036`.
- Student email: `231a290036@vhu.edu.vn`.
- The profile card does not display a class line.
- Android min SDK is 24.
- UI dimensions use dp and text sizes use sp; visible copy comes from `strings.xml`.
- Portrait, landscape, and ConstraintLayout variants are all required.
- Advanced tasks are NC1 (`values-night/colors.xml`) and NC3 (reusable text styles).
- The repository is `VOVANQUOCBAO/homework`.

## Review Focus

1. Landscape resource selection must not crash: every ID read by `MainActivity` is present in both `layout/activity_main.xml` variants; Task 3 pins this with `test_main_activity_ids_match_across_orientations`.
2. Small screens and open keyboards must retain access to content: portrait and both landscape columns use scroll containers; Task 2 and Task 3 pin this with scroll assertions.
3. ConstraintLayout views must not jump to coordinate 0,0: every direct visual child has horizontal and vertical constraints; Task 3 pins this with `test_constraint_children_have_both_axes`.
4. Dark mode must not fall back to incomplete colors: the night file defines the same named color set used by layouts/themes; Task 1 pins this with `test_night_palette_covers_day_palette`.
5. Accidental hardcoded Vietnamese copy must not evade localization: visible-text attributes reference string resources; Task 2 pins this with `test_visible_text_uses_string_resources`.

---

### Task 1: Project foundation and shared resources

**Files:**
- Create: `.gitignore`
- Create: `settings.gradle.kts`
- Create: `build.gradle.kts`
- Create: `gradle.properties`
- Create: `app/build.gradle.kts`
- Create: `app/proguard-rules.pro`
- Create: `app/src/main/AndroidManifest.xml`
- Create: `app/src/main/res/values/strings.xml`
- Create: `app/src/main/res/values/colors.xml`
- Create: `app/src/main/res/values-night/colors.xml`
- Create: `app/src/main/res/values/dimens.xml`
- Create: `app/src/main/res/values/themes.xml`
- Create: `app/src/main/res/values-night/themes.xml`
- Create: `app/src/main/res/drawable/bg_header.xml`
- Create: `app/src/main/res/drawable/bg_avatar.xml`
- Create: `app/src/main/res/drawable/bg_stat.xml`
- Create: `tests/test_project_structure.py`

**Interfaces:**
- Consumes: identity and version constraints from the spec.
- Produces: Android module `:app`; resources `@string/*`, `@color/*`, `@dimen/*`, `@style/TitleText`, `@style/SubtitleText`, and drawable shapes used by later tasks.

- [ ] **Step 1: Write the failing foundation tests**

Create `tests/test_project_structure.py` with helpers and these tests:

```python
from pathlib import Path
import re
import xml.etree.ElementTree as ET

ROOT = Path(__file__).resolve().parents[1]
RES = ROOT / "app/src/main/res"
ANDROID = "{http://schemas.android.com/apk/res/android}"

def names(path: Path, tag: str) -> set[str]:
    return {node.attrib["name"] for node in ET.parse(path).getroot().findall(tag)}

def test_project_identity_and_min_sdk():
    gradle = (ROOT / "app/build.gradle.kts").read_text(encoding="utf-8")
    strings = ET.parse(RES / "values/strings.xml").getroot()
    values = {node.attrib["name"]: node.text for node in strings.findall("string")}
    assert 'namespace = "vn.edu.vhu.ltdd.a3layout"' in gradle
    assert "minSdk = 24" in gradle
    assert values["app_name"] == "A3_231A290036"
    assert values["student_name"] == "Võ Văn Quốc Bảo"
    assert values["student_id_line"] == "MSSV: 231A290036"
    assert values["email_line"] == "231a290036@vhu.edu.vn"
    assert "class_line" not in values

def test_night_palette_covers_day_palette():
    assert names(RES / "values/colors.xml", "color") == names(
        RES / "values-night/colors.xml", "color"
    )

def test_drawables_are_vector_independent_shapes():
    for filename in ("bg_header.xml", "bg_avatar.xml", "bg_stat.xml"):
        assert ET.parse(RES / "drawable" / filename).getroot().tag == "shape"

def test_reusable_text_styles_exist():
    styles = names(RES / "values/themes.xml", "style")
    assert {"TitleText", "SubtitleText"} <= styles
```

- [ ] **Step 2: Run tests and verify the expected failure**

Run: `python3 -m unittest discover -s tests -v`

Expected: ERROR/FAIL because `app/build.gradle.kts` and resource files do not exist.

- [ ] **Step 3: Implement the project and resources**

Create the Gradle files with the versions in the plan header. Configure `compileSdk = 35`, `minSdk = 24`, `targetSdk = 35`, Java 17, dependencies for AppCompat, Material, ConstraintLayout, and JUnit 4.13.2. Create a Material3 no-action-bar theme, exact identity strings, all copy from the lab, day/night palettes with identical names, spacing dimensions, the two reusable text styles, and the three shape drawables. Declare `MainActivity` as the exported launcher and `ConstraintDemoActivity` as non-exported.

- [ ] **Step 4: Run foundation tests**

Run: `python3 -m unittest discover -s tests -v`

Expected: all four tests PASS.

- [ ] **Step 5: Commit the foundation**

```bash
git add .gitignore settings.gradle.kts build.gradle.kts gradle.properties app tests
git commit -m "build: scaffold Android app and shared resources"
```

### Task 2: Portrait login layout and reusable profile card

**Files:**
- Create: `app/src/main/res/layout/view_profile_card.xml`
- Create: `app/src/main/res/layout/activity_main.xml`
- Modify: `tests/test_project_structure.py`

**Interfaces:**
- Consumes: strings, colors, dimensions, styles, and drawables from Task 1.
- Produces: portrait IDs `main`, `edtStudentId`, `edtPassword`, `cbRemember`, `btnForgot`, `btnLogin`, `btnSchoolLogin`, `btnRegister`, `cardProfile`, and `btnConstraintDemo` for MainActivity and landscape parity.

- [ ] **Step 1: Add failing portrait tests**

Append:

```python
def parsed(relative: str):
    return ET.parse(RES / relative).getroot()

def test_portrait_contains_required_layout_mechanics():
    root = parsed("layout/activity_main.xml")
    xml = ET.tostring(root, encoding="unicode")
    assert root.tag == "ScrollView"
    assert "FrameLayout" in xml
    assert "Space" in xml
    assert "layout_weight" in xml
    assert '@layout/view_profile_card' in xml
    assert '@drawable/bg_header' in xml
    assert '@drawable/bg_avatar' in xml

def test_profile_omits_class_and_uses_identity_resources():
    xml = (RES / "layout/view_profile_card.xml").read_text(encoding="utf-8")
    assert "@string/student_name" in xml
    assert "@string/student_id_line" in xml
    assert "@string/email_line" in xml
    assert "class_line" not in xml

def test_visible_text_uses_string_resources():
    for path in (RES / "layout").glob("*.xml"):
        for node in ET.parse(path).iter():
            value = node.attrib.get(ANDROID + "text")
            if value is not None:
                assert value.startswith("@string/"), f"{path}: {value}"
```

- [ ] **Step 2: Run portrait tests and verify failure**

Run: `python3 -m unittest tests.test_project_structure -v`

Expected: ERROR because the portrait and profile XML files do not exist.

- [ ] **Step 3: Implement portrait and profile XML**

Create the profile as a `MaterialCardView` containing name, MSSV, email, divider, and two equal-width statistics with `layout_width="0dp"` plus `layout_weight="1"`. Create the portrait root as `ScrollView` with `fillViewport="true"`; place a vertical LinearLayout inside it. Use a 196dp FrameLayout, 150dp gradient header, 92dp avatar, TextInputLayout fields, weighted Space, weighted divider views, outlined school-login button, profile include, and ConstraintLayout navigation button. Apply `TitleText` and `SubtitleText`.

- [ ] **Step 4: Run portrait tests**

Run: `python3 -m unittest tests.test_project_structure -v`

Expected: all seven accumulated tests PASS.

- [ ] **Step 5: Commit portrait UI**

```bash
git add app/src/main/res/layout tests/test_project_structure.py
git commit -m "feat: build portrait login and profile layouts"
```

### Task 3: Java behavior, landscape variant, and ConstraintLayout variant

**Files:**
- Create: `app/src/main/java/vn/edu/vhu/ltdd/a3layout/MainActivity.java`
- Create: `app/src/main/java/vn/edu/vhu/ltdd/a3layout/ConstraintDemoActivity.java`
- Create: `app/src/main/res/layout-land/activity_main.xml`
- Create: `app/src/main/res/layout/activity_constraint_demo.xml`
- Modify: `tests/test_project_structure.py`

**Interfaces:**
- Consumes: portrait ID contract from Task 2 and resources from Task 1.
- Produces: executable launcher behavior, automatic two-column landscape UI, and navigable ConstraintLayout demo.

- [ ] **Step 1: Add failing behavior and variant tests**

Append:

```python
def ids(path: Path) -> set[str]:
    found = set()
    for node in ET.parse(path).iter():
        value = node.attrib.get(ANDROID + "id", "")
        if value.startswith("@+id/"):
            found.add(value.removeprefix("@+id/"))
    return found

def test_main_activity_ids_match_across_orientations():
    required = {"main", "edtStudentId", "edtPassword", "cbRemember",
                "btnForgot", "btnLogin", "btnSchoolLogin", "btnRegister",
                "cardProfile", "btnConstraintDemo"}
    portrait = ids(RES / "layout/activity_main.xml")
    landscape = ids(RES / "layout-land/activity_main.xml")
    assert required <= portrait
    assert required <= landscape

def test_landscape_is_two_column_and_scrollable():
    xml = (RES / "layout-land/activity_main.xml").read_text(encoding="utf-8")
    assert 'android:orientation="horizontal"' in xml
    assert xml.count("layout_weight") >= 4
    assert xml.count("<ScrollView") == 2

def test_constraint_children_have_both_axes():
    root = parsed("layout/activity_constraint_demo.xml")
    app = "{http://schemas.android.com/apk/res-auto}"
    for node in list(root):
        if node.tag.endswith("Guideline"):
            continue
        keys = node.attrib
        horizontal = any(k.startswith(app + "layout_constraintStart_") or
                         k.startswith(app + "layout_constraintEnd_") for k in keys)
        vertical = any(k.startswith(app + "layout_constraintTop_") or
                       k.startswith(app + "layout_constraintBottom_") or
                       k == app + "layout_constraintBaseline_toBaselineOf" for k in keys)
        assert horizontal and vertical, node.attrib.get(ANDROID + "id", node.tag)

def test_java_wires_snackbar_and_constraint_navigation():
    source = (ROOT / "app/src/main/java/vn/edu/vhu/ltdd/a3layout/MainActivity.java").read_text(encoding="utf-8")
    assert 'TAG = "A3_231A290036"' in source
    assert "Snackbar.make" in source
    assert "cbRemember.isChecked()" in source
    assert "new Intent(this, ConstraintDemoActivity.class)" in source
```

- [ ] **Step 2: Run variant tests and verify failure**

Run: `python3 -m unittest tests.test_project_structure -v`

Expected: ERROR because landscape, ConstraintLayout, and Java files do not exist.

- [ ] **Step 3: Implement activities and alternate layouts**

Implement `MainActivity.onCreate(Bundle)` with EdgeToEdge, insets for `R.id.main`, orientation logging, Snackbar behavior, and navigation. Implement `ConstraintDemoActivity.onCreate(Bundle)` with `setContentView(R.layout.activity_constraint_demo)` and title. Build the landscape 2:3 weighted layout with two ScrollViews and all required IDs. Build the ConstraintLayout with two vertical Guidelines, full horizontal and vertical constraints on every visual child, match-constraint widths, and a two-button horizontal chain.

- [ ] **Step 4: Run complete structural test suite**

Run: `python3 -m unittest discover -s tests -v`

Expected: all eleven tests PASS with no warnings or errors.

- [ ] **Step 5: Run Android verification**

Run: `./gradlew test lintDebug assembleDebug`

Expected: `BUILD SUCCESSFUL`; unit tests, lint, resource linking, and Java compilation complete successfully.

- [ ] **Step 6: Commit executable application**

```bash
git add app/src/main tests/test_project_structure.py
git commit -m "feat: add landscape and ConstraintLayout demos"
```

### Task 4: Submission documentation and final verification

**Files:**
- Create: `README.md`
- Create: `docs/BAO_CAO_LAB_A3.md`
- Modify: `tests/test_project_structure.py`

**Interfaces:**
- Consumes: the completed app and test evidence from Tasks 1-3.
- Produces: opening/build instructions, advanced-work explanation, layout comparison, five review answers, and a truthful capture/video checklist.

- [ ] **Step 1: Add failing documentation tests**

Append:

```python
def test_submission_docs_cover_required_sections():
    readme = (ROOT / "README.md").read_text(encoding="utf-8")
    report = (ROOT / "docs/BAO_CAO_LAB_A3.md").read_text(encoding="utf-8")
    assert "231A290036" in readme
    assert "NC1" in readme and "NC3" in readme
    assert "LinearLayout" in report and "ConstraintLayout" in report
    for number in range(1, 6):
        assert f"### Câu {number}" in report
    assert "Ảnh cần bổ sung trước khi nộp" in report
    assert "Video demo" in report
```

- [ ] **Step 2: Run documentation test and verify failure**

Run: `python3 -m unittest tests.test_project_structure.ProjectStructureTests.test_submission_docs_cover_required_sections -v` if tests are class-based; otherwise run `python3 -m unittest discover -s tests -v`.

Expected: ERROR because README and report do not exist.

- [ ] **Step 3: Write submission documentation**

Create README with project identity, prerequisites, Android Studio run instructions, project structure, NC1/NC3 evidence, test command, and demo checklist. Create the report with student details, comparison table, answers explaining padding/margin, gravity/layout_gravity, dp/sp/px, weighted equal-width buttons, reasons for flattening layouts, qualifier selection/activity recreation, and clearly labeled slots the student must replace with real hand-drawn/emulator captures. Include a two-minute demo script.

- [ ] **Step 4: Run final verification from a clean state**

Run:

```bash
python3 -m unittest discover -s tests -v
./gradlew clean test lintDebug assembleDebug
git status --short
git log --oneline --decorate -5
```

Expected: all structural tests PASS, Gradle prints `BUILD SUCCESSFUL`, Git status lists only intentional documentation/test changes before the final commit, and history contains the design, plan, foundation, portrait, and executable-app commits.

- [ ] **Step 5: Commit documentation**

```bash
git add README.md docs/BAO_CAO_LAB_A3.md tests/test_project_structure.py
git commit -m "docs: add Lab A3 report and demo checklist"
```

- [ ] **Step 6: Verify committed state and push**

Run:

```bash
python3 -m unittest discover -s tests -v
./gradlew test lintDebug assembleDebug
git status --short
git log --oneline --decorate -7
git push origin main
```

Expected: tests and build succeed, working tree is clean, at least six meaningful commits exist, and `main` is pushed to `VOVANQUOCBAO/homework`.
