---
id: VS-001
challenge: challenges/web/challenge-001.md
---

# Solution VS-001 — Flask Template Trap

## ✅ Vulnerability / Vulnerabilidad

**Server-Side Template Injection (SSTI) — CWE-94**

Jinja2 template engine processes attacker-controlled input as executable template code.

---

## 🔎 Explanation / Explicación

The vulnerability lives in the interaction between Python's `.format()` and Flask's `render_template_string()`.

Here's the critical sequence:

```python
# Step 1: Python string interpolation — user input dropped directly into template string
template = REPORT_WRAPPER.format(user_content=report_notes)

# Step 2: The resulting string is passed to Jinja2 for rendering
return render_template_string(template)
```

`REPORT_WRAPPER.format(user_content=report_notes)` runs first, embedding the raw user input into the HTML string as literal text. The output of that is then passed to `render_template_string()`, which processes it as a Jinja2 template.

If `report_notes` contains Jinja2 expressions like `{{ 7*7 }}`, they are inserted into the template *before* Jinja2 renders it. Jinja2 then evaluates them as code.

A developer reviewing this code might assume the `REPORT_WRAPPER` template controls what gets executed. It does — until user input is written into it before rendering.

**Proof of Concept payload:**
```
{{ config.__class__.__init__.__globals__['os'].popen('id').read() }}
```

This uses Jinja2's Python object access to call `os.popen()` and read the output — achieving Remote Code Execution.

[ES] La vulnerabilidad reside en la interacción entre `.format()` de Python y `render_template_string()` de Flask.

`REPORT_WRAPPER.format(user_content=report_notes)` se ejecuta primero, insertando el input crudo del usuario en la cadena HTML como texto literal. El resultado se pasa a `render_template_string()`, que lo procesa como un template de Jinja2.

Si `report_notes` contiene expresiones Jinja2 como `{{ 7*7 }}`, se insertan en el template *antes* de que Jinja2 lo renderice. Jinja2 las evalúa como código.

Un desarrollador revisando el código podría asumir que el template `REPORT_WRAPPER` controla qué se ejecuta. Lo hace — hasta que el input del usuario se escribe en él antes del renderizado.

---

## 💥 Impact / Impacto

Full Remote Code Execution (RCE) in the context of the web server process. An attacker can:

- Read files from the filesystem (credentials, environment variables, SSH keys)
- Execute arbitrary OS commands
- Pivot to other internal systems from the server
- Exfiltrate application secrets (`app.secret_key`, database credentials in environment)

"Internal only" does not mitigate this — any user who can reach the form can exploit it.

---

## 🛡️ Remediation / Remediación

**Never pass user input through `render_template_string()`.** Use a static template file and pass data as context variables. Jinja2's autoescaping handles output encoding, but it cannot protect you if the user controls the *template itself*.

**Fixed version:**

```python
# templates/report.html — static file, not constructed from user input
# <!DOCTYPE html>
# <html><body>
#   <h2>Internal Status Report</h2>
#   <div class="report-body">{{ report_notes }}</div>
# </body></html>

from flask import Flask, request, render_template

app = Flask(__name__)

@app.route("/report", methods=["GET", "POST"])
def generate_report():
    if request.method == "POST":
        engineer_name = request.form.get("name", "Unknown")
        report_notes = request.form.get("notes", "")

        # User data passed as context, NOT embedded in the template string
        return render_template("report.html", report_notes=report_notes, name=engineer_name)

    return """
        <form method="POST">
            Name: <input name="name"><br>
            Notes: <textarea name="notes"></textarea><br>
            <input type="submit" value="Generate Report">
        </form>
    """
```

With `render_template()`, user content is treated as data, not code. Jinja2 autoescape handles the HTML encoding. The template engine never sees user input as template syntax.

---

## 📚 References / Referencias

- [OWASP: Server-Side Template Injection](https://owasp.org/www-project-web-security-testing-guide/v42/4-Web_Application_Security_Testing/07-Input_Validation_Testing/18-Testing_for_Server-side_Template_Injection)
- [CWE-94: Improper Control of Generation of Code](https://cwe.mitre.org/data/definitions/94.html)
- [PortSwigger: SSTI](https://portswigger.net/web-security/server-side-template-injection)
- [MITRE ATT&CK T1059.006: Python](https://attack.mitre.org/techniques/T1059/006/)
