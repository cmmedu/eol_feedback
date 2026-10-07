# EOL Feedback

![Coverage Status](/coverage-badge.svg)

![https://github.com/eol-uchile/eol_feedback/actions](https://github.com/eol-uchile/eol_feedback/workflows/Python%20application%20test/badge.svg) 

Adding feedback for student exams

## Configurations

Migrate Models:

    > docker-compose exec lms python manage.py lms --settings=tutor.production migrate eol_feedback

LMS Django Admin:

- */admin/site_configuration/siteconfiguration/*
    - **"EOL_FEEDBACK_TAB_ENABLED":true**
- */admin/grades/persistentgradesenabledflag/*
    - Set Flag Enabled
- */admin/grades/coursepersistentgradesflag/*
    - Add course id
    - Set Enable
- */admin/waffle/switch/*
    - Add Switch and set Enable: **grades.assume_zero_grade_if_absent**

If courses already exists:

    > docker-compose exec lms python manage.py lms --settings=tutor.production compute_grades --courses COURSE_ID

## Subsection formats (per course)

The feedback page groups subsections by their **assignment type** (the "format" of the subsection):

- **Graded questions** (default: `Preguntas Evaluadas`, `Preguntas Calificadas`): averaged into one block per section.
- **Controls** (default: `Control`): one block per subsection, with correct/incorrect answers and feedback.

A subsection matches when its format **contains** any of the configured strings, ignoring case
(e.g. `Control` matches `Control 1` and `Control del Taller 7`). Each course can override the defaults.

**1. Enable "Other Course Settings" in Studio**

Add the feature flag to the CMS config (`cms.yml`, or `cms.env.json` / tutor `cms.yml` depending on the deployment), under `FEATURES`:

    FEATURES:
      ENABLE_OTHER_COURSE_SETTINGS: true

Restart the CMS. The field **Other Course Settings** now appears in the course's *Settings > Advanced Settings*.

**2. Enable the tab in the Studio site configuration too**

Saving Advanced Settings refreshes the course tabs using the **Studio** site configuration. If the flag is only set
in the LMS site configuration, the Feedback tab is removed from the course when Advanced Settings are saved.

- CMS site in */admin/site_configuration/siteconfiguration/*
    - **"EOL_FEEDBACK_TAB_ENABLED":true**

If the tab already disappeared, set the flag and save Advanced Settings again (no changes needed) to restore it.

**3. Create the assignment types**

In Studio, *Settings > Grading*, create the assignment types used by the course, e.g.
`Preguntas Calificadas Taller 1`, `Control 1`.

**4. Assign the type to each question subsection**

In the course outline, open the subsection settings (gear icon) > *Grading* > *Grade as*, and select the assignment
type (e.g. `Preguntas Calificadas Taller 1` for a graded question, `Control 1` for a control).

**5. (Optional) Set the formats for the course**

In *Settings > Advanced Settings > Other Course Settings*, add the `eol_feedback` key:

    {
        "eol_feedback": {
            "graded_formats": ["Preguntas Evaluadas", "Preguntas Calificadas"],
            "control_formats": ["Control"]
        }
    }

- Each key accepts a list of strings or a single string.
- A missing, empty or invalid key falls back to the default value.

**6. Enable the feedback for each section**

As staff, open the **Feedback** tab and click *Retroalimentación desactivada* on each section to enable it.
Disabled sections are not shown to students. Use *View this course as: Learner* to check the student view.

## TESTS
**Prepare tests:**

    > cd .github/
    > docker-compose run lms /openedx/requirements/eol_feedback/.github/test.sh


## TO DO:

- [ ] Tab has to be initialized on course creation. In this moment the tab will appear after first changes on Advanced settings.
- [ ] Final grade calculated by percentage of progress. If the student didn't answer a test (or the test is not released) he will have 0% (0.0) and final grade will be affected. See what to do in this case.
