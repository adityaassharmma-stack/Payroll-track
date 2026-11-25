/* =====================================================
   AUTOMATED PAYROLL MANAGEMENT & ANALYTICS SYSTEM
   Beginner-friendly JavaScript
   Storage: Browser Local Storage
   ===================================================== */


/* 1. GET ELEMENTS AND LOAD DATA */

function getElement(id) {
    return document.getElementById(id);
}

let employees = JSON.parse(localStorage.getItem("payrollEmployees")) || [];
let payrollRecords = JSON.parse(localStorage.getItem("payrollRecords")) || [];

let settings = JSON.parse(localStorage.getItem("payrollSettings")) || {
    companyName: "My Company",
    currency: "INR"
};

let currentPayslipId = null;


/* 2. COMMON FUNCTIONS */

function saveData() {
    localStorage.setItem("payrollEmployees", JSON.stringify(employees));
    localStorage.setItem("payrollRecords", JSON.stringify(payrollRecords));
    localStorage.setItem("payrollSettings", JSON.stringify(settings));
}

function formatCurrency(amount) {
    let symbol = settings.currency === "INR" ? "₹" : settings.currency + " ";
    return symbol + Number(amount || 0).toLocaleString("en-IN", {
        maximumFractionDigits: 2
    });
}

function showMessage(message) {
    let toast = getElement("toast");

    if (toast) {
        toast.textContent = message;
        toast.classList.add("show");

        setTimeout(function () {
            toast.classList.remove("show");
        }, 2500);
    } else {
        alert(message);
    }
}

function escapeHTML(value) {
    return String(value ?? "").replace(/[&<>"']/g, function (character) {
        let characters = {
            "&": "&amp;",
            "<": "&lt;",
            ">": "&gt;",
            '"': "&quot;",
            "'": "&#039;"
        };

        return characters[character];
    });
}

function getNumber(id) {
    return Number(getElement(id).value) || 0;
}

function findEmployee(id) {
    for (let i = 0; i < employees.length; i++) {
        if (employees[i].id === id) {
            return employees[i];
        }
    }

    return null;
}

function findPayroll(id) {
    for (let i = 0; i < payrollRecords.length; i++) {
        if (String(payrollRecords[i].id) === String(id)) {
            return payrollRecords[i];
        }
    }

    return null;
}


/* 3. NAVIGATION BETWEEN MODULES */

let navigationLinks = document.querySelectorAll(".nav-link");

navigationLinks.forEach(function (link) {
    link.addEventListener("click", function () {
        let sectionName = link.getAttribute("data-section");

        document.querySelectorAll(".content-section").forEach(function (section) {
            section.classList.remove("active");
        });

        let selectedSection = getElement(sectionName);

        if (selectedSection) {
            selectedSection.classList.add("active");
        }

        navigationLinks.forEach(function (item) {
            item.classList.remove("active");
        });

        link.classList.add("active");

        let pageTitle = getElement("page-title");

        if (pageTitle) {
            pageTitle.textContent = link.textContent.trim();
        }
    });
});

document.querySelectorAll("[data-go]").forEach(function (button) {
    button.addEventListener("click", function () {
        let sectionName = button.getAttribute("data-go");
        let link = document.querySelector(
            '.nav-link[data-section="' + sectionName + '"]'
        );

        if (link) {
            link.click();
        }
    });
});


/* 4. EMPLOYEE MANAGEMENT */

function resetEmployeeForm() {
    getElement("employee-form").reset();
    getElement("edit-employee-id").value = "";
    getElement("employee-form-heading").textContent = "Add Employee";
    getElement("employee-submit-button").textContent = "Add Employee";
    getElement("cancel-edit-button").style.display = "none";
}

function addOrUpdateEmployee(event) {
    event.preventDefault();

    let editId = getElement("edit-employee-id").value;

    let employee = {
        id: getElement("employee-id").value.trim(),
        name: getElement("employee-name").value.trim(),
        department: getElement("employee-department").value,
        designation: getElement("employee-designation").value.trim(),
        basicSalary: getNumber("employee-basic"),
        hra: getNumber("employee-hra"),
        allowance: getNumber("employee-allowance"),
        pf: getNumber("employee-pf"),
        otherDeduction: getNumber("employee-other-deduction")
    };

    if (!employee.id || !employee.name) {
        showMessage("Please enter employee ID and name.");
        return;
    }

    if (editId === "") {
        if (findEmployee(employee.id)) {
            showMessage("Employee ID already exists.");
            return;
        }

        employees.push(employee);
        showMessage("Employee added successfully.");
    } else {
        for (let i = 0; i < employees.length; i++) {
            if (employees[i].id === editId) {
                employees[i] = employee;
                break;
            }
        }

        showMessage("Employee updated successfully.");
    }

    saveData();
    resetEmployeeForm();
    refreshApplication();
}

function editEmployee(id) {
    let employee = findEmployee(id);

    if (!employee) {
        return;
    }

    getElement("edit-employee-id").value = employee.id;
    getElement("employee-id").value = employee.id;
    getElement("employee-name").value = employee.name;
    getElement("employee-department").value = employee.department;
    getElement("employee-designation").value = employee.designation;
    getElement("employee-basic").value = employee.basicSalary;
    getElement("employee-hra").value = employee.hra;
    getElement("employee-allowance").value = employee.allowance;
    getElement("employee-pf").value = employee.pf;
    getElement("employee-other-deduction").value = employee.otherDeduction;

    getElement("employee-id").readOnly = true;
    getElement("employee-form-heading").textContent = "Edit Employee";
    getElement("employee-submit-button").textContent = "Update Employee";
    getElement("cancel-edit-button").style.display = "inline-block";

    getElement("employee-form").scrollIntoView({
        behavior: "smooth"
    });
}

function deleteEmployee(id) {
    let confirmDelete = confirm("Are you sure you want to delete this employee?");

    if (!confirmDelete) {
        return;
    }

    employees = employees.filter(function (employee) {
        return employee.id !== id;
    });

    saveData();
    refreshApplication();
    showMessage("Employee deleted.");
}

function displayEmployees() {
    let table = getElement("employee-table-body");

    if (!table) {
        return;
    }

    let searchText = getElement("employee-search").value.toLowerCase();
    let selectedDepartment = getElement("department-filter").value;

    let count = 0;
    let html = "";

    for (let i = 0; i < employees.length; i++) {
        let employee = employees[i];

        let matchesSearch =
            employee.name.toLowerCase().includes(searchText) ||
            employee.id.toLowerCase().includes(searchText);

        let matchesDepartment =
            selectedDepartment === "" ||
            employee.department === selectedDepartment;

        if (matchesSearch && matchesDepartment) {
            count++;

            html += `
                <tr>
                    <td>${escapeHTML(employee.id)}</td>
                    <td>${escapeHTML(employee.name)}</td>
                    <td>${escapeHTML(employee.department)}</td>
                    <td>${escapeHTML(employee.designation)}</td>
                    <td>${formatCurrency(employee.basicSalary)}</td>
                    <td>
                        <button class="edit-btn" data-edit="${escapeHTML(employee.id)}">
                            Edit
                        </button>
                        <button class="delete-btn" data-delete="${escapeHTML(employee.id)}">
                            Delete
                        </button>
                    </td>
                </tr>
            `;
        }
    }

    table.innerHTML = html || '<tr><td colspan="6">No employees found.</td></tr>';

    getElement("employee-count-label").textContent =
        count + " employees";
}

getElement("employee-form").addEventListener("submit", addOrUpdateEmployee);

getElement("cancel-edit-button").addEventListener("click", function () {
    resetEmployeeForm();
    getElement("employee-id").readOnly = false;
});

getElement("employee-search").addEventListener("input", displayEmployees);
getElement("department-filter").addEventListener("change", displayEmployees);

getElement("employee-table-body").addEventListener("click", function (event) {
    let editId = event.target.getAttribute("data-edit");
    let deleteId = event.target.getAttribute("data-delete");

    if (editId) {
        editEmployee(editId);
    }

    if (deleteId) {
        deleteEmployee(deleteId);
    }
});


/* 5. PAYROLL PROCESSING */

function fillEmployeeSalary() {
    let employeeId = getElement("payroll-employee").value;
    let employee = findEmployee(employeeId);

    if (!employee) {
        return;
    }

    getElement("payroll-basic").value = employee.basicSalary;
    getElement("payroll-hra").value = employee.hra;
    getElement("payroll-allowance").value = employee.allowance;
    getElement("payroll-pf").value = employee.pf;
    getElement("payroll-other-deduction").value = employee.otherDeduction;
}

function calculateSalary() {
    let basic = getNumber("payroll-basic");
    let hra = getNumber("payroll-hra");
    let allowance = getNumber("payroll-allowance");
    let pf = getNumber("payroll-pf");
    let other = getNumber("payroll-other-deduction");

    let gross = basic + hra + allowance;
    let deductions = pf + other;
    let net = gross - deductions;

    getElement("result-gross").textContent = formatCurrency(gross);
    getElement("result-deductions").textContent = formatCurrency(deductions);
    getElement("result-net").textContent = formatCurrency(net);

    return {
        gross: gross,
        deductions: deductions,
        net: net
    };
}

function processPayroll(event) {
    event.preventDefault();

    let employeeId = getElement("payroll-employee").value;
    let employee = findEmployee(employeeId);
    let month = getElement("payroll-month").value;

    if (!employee || !month) {
        showMessage("Please select an employee and payroll month.");
        return;
    }

    for (let i = 0; i < payrollRecords.length; i++) {
        if (
            payrollRecords[i].employeeId === employeeId &&
            payrollRecords[i].month === month
        ) {
            showMessage("Payroll already exists for this employee and month.");
            return;
        }
    }

    let salary = calculateSalary();

    if (salary.net < 0) {
        showMessage("Deductions cannot be greater than gross salary.");
        return;
    }

    let record = {
        id: Date.now(),
        employeeId: employee.id,
        employeeName: employee.name,
        department: employee.department,
        designation: employee.designation,
        month: month,
        basic: getNumber("payroll-basic"),
        hra: getNumber("payroll-hra"),
        allowance: getNumber("payroll-allowance"),
        pf: getNumber("payroll-pf"),
        otherDeduction: getNumber("payroll-other-deduction"),
        gross: salary.gross,
        deductions: salary.deductions,
        net: salary.net,
        date: new Date().toLocaleDateString("en-IN")
    };

    payrollRecords.push(record);

    saveData();
    refreshApplication();

    showMessage("Payroll processed successfully.");
}

getElement("payroll-employee").addEventListener("change", fillEmployeeSalary);
getElement("calculate-button").addEventListener("click", calculateSalary);
getElement("payroll-form").addEventListener("submit", processPayroll);


/* 6. PAYSLIP GENERATION */

function displayPayslipTable() {
    let table = getElement("payslip-table-body");
    let html = "";

    for (let i = 0; i < payrollRecords.length; i++) {
        let record = payrollRecords[i];

        html += `
            <tr>
                <td>${escapeHTML(record.employeeName)}</td>
                <td>${escapeHTML(record.month)}</td>
                <td>${formatCurrency(record.net)}</td>
                <td>
                    <button class="view-payslip-btn" data-payslip="${record.id}">
                        View Slip
                    </button>
                </td>
            </tr>
        `;
    }

    table.innerHTML = html ||
        '<tr><td colspan="4">No payroll records available.</td></tr>';
}

function generatePayslip(id) {
    let record = findPayroll(id);

    if (!record) {
        return;
    }

    currentPayslipId = record.id;

    let preview = getElement("payslip-preview");

    preview.innerHTML = `
        <div class="payslip-content">
            <h2>${escapeHTML(settings.companyName)}</h2>
            <h3>Employee Salary Slip</h3>
            <hr>

            <p><strong>Employee ID:</strong> ${escapeHTML(record.employeeId)}</p>
            <p><strong>Employee Name:</strong> ${escapeHTML(record.employeeName)}</p>
            <p><strong>Department:</strong> ${escapeHTML(record.department)}</p>
            <p><strong>Designation:</strong> ${escapeHTML(record.designation)}</p>
            <p><strong>Salary Month:</strong> ${escapeHTML(record.month)}</p>

            <hr>
            <h3>Salary Details</h3>

            <p>Basic Salary: ${formatCurrency(record.basic)}</p>
            <p>HRA: ${formatCurrency(record.hra)}</p>
            <p>Allowances: ${formatCurrency(record.allowance)}</p>
            <p><strong>Gross Salary: ${formatCurrency(record.gross)}</strong></p>

            <hr>
            <p>Provident Fund: ${formatCurrency(record.pf)}</p>
            <p>Other Deductions: ${formatCurrency(record.otherDeduction)}</p>
            <p><strong>Total Deductions: ${formatCurrency(record.deductions)}</strong></p>

            <hr>
            <h2>Net Salary: ${formatCurrency(record.net)}</h2>
            <p>Generated on: ${escapeHTML(record.date)}</p>
        </div>

        <button type="button" id="print-payslip-button">Print / Save as PDF</button>
    `;

    getElement("print-payslip-button").addEventListener("click", function () {
        window.print();
    });
}

getElement("payslip-table-body").addEventListener("click", function (event) {
    let id = event.target.getAttribute("data-payslip");

    if (id) {
        generatePayslip(id);
    }
});


/* 7. DASHBOARD */

function updateDashboard() {
    let totalGross = 0;
    let totalDeductions = 0;
    let totalNet = 0;

    for (let i = 0; i < payrollRecords.length; i++) {
        totalGross += payrollRecords[i].gross;
        totalDeductions += payrollRecords[i].deductions;
        totalNet += payrollRecords[i].net;
    }

    getElement("total-employees").textContent = employees.length;
    getElement("total-gross").textContent = formatCurrency(totalGross);
    getElement("total-deductions").textContent = formatCurrency(totalDeductions);
    getElement("total-net").textContent = formatCurrency(totalNet);

    let recentTable = getElement("recent-payroll-body");
    let html = "";

    for (let i = payrollRecords.length - 1; i >= 0 && i >= payrollRecords.length - 5; i--) {
        let record = payrollRecords[i];

        html += `
            <tr>
                <td>${escapeHTML(record.employeeName)}</td>
                <td>${escapeHTML(record.month)}</td>
                <td>${formatCurrency(record.net)}</td>
            </tr>
        `;
    }

    recentTable.innerHTML = html ||
        '<tr><td colspan="3">No recent payroll activity.</td></tr>';

    getElement("today-date").textContent =
        new Date().toLocaleDateString("en-IN", {
            day: "numeric",
            month: "long",
            year: "numeric"
        });
}


/* 8. ANALYTICS AND CHARTS */

function updateAnalytics() {
    getElement("analytics-employee-count").textContent = employees.length;
    getElement("analytics-record-count").textContent = payrollRecords.length;

    let totalNet = 0;

    for (let i = 0; i < payrollRecords.length; i++) {
        totalNet += payrollRecords[i].net;
    }

    let average = payrollRecords.length > 0
        ? totalNet / payrollRecords.length
        : 0;

    getElement("analytics-average").textContent = formatCurrency(average);

    drawDepartmentChart();
    drawSalaryChart();
    drawMonthlyChart();
}

function drawDepartmentChart() {
    let totals = {};

    for (let i = 0; i < payrollRecords.length; i++) {
        let department = payrollRecords[i].department;

        if (!totals[department]) {
            totals[department] = 0;
        }

        totals[department] += payrollRecords[i].net;
    }

    let chart = getElement("department-chart");
    let html = "";
    let departments = Object.keys(totals);

    for (let i = 0; i < departments.length; i++) {
        let name = departments[i];
        let amount = totals[name];
        let maximum = Math.max(...Object.values(totals));
        let width = maximum > 0 ? (amount / maximum) * 100 : 0;

        html += `
            <div class="bar-item">
                <span>${escapeHTML(name)}</span>
                <div class="bar" style="width:${width}%"></div>
                <strong>${formatCurrency(amount)}</strong>
            </div>
        `;
    }

    chart.innerHTML = html || "<p>No department data available.</p>";

    getElement("analytics-departments").textContent =
        departments.length;
}

function drawSalaryChart() {
    let gross = 0;
    let net = 0;

    for (let i = 0; i < payrollRecords.length; i++) {
        gross += payrollRecords[i].gross;
        net += payrollRecords[i].net;
    }

    let chart = getElement("salary-chart");
    let maximum = Math.max(gross, 1);

    chart.innerHTML = `
        <div class="bar-item">
            <span>Gross Salary</span>
            <div class="bar" style="width:${gross / maximum * 100}%"></div>
            <strong>${formatCurrency(gross)}</strong>
        </div>
        <div class="bar-item">
            <span>Net Salary</span>
            <div class="bar" style="width:${net / maximum * 100}%"></div>
            <strong>${formatCurrency(net)}</strong>
        </div>
    `;
}

function drawMonthlyChart() {
    let totals = {};

    for (let i = 0; i < payrollRecords.length; i++) {
        let month = payrollRecords[i].month;

        if (!totals[month]) {
            totals[month] = 0;
        }

        totals[month] += payrollRecords[i].net;
    }

    let chart = getElement("monthly-chart");
    let months = Object.keys(totals);
    let html = "";
    let maximum = Math.max(1, ...Object.values(totals));

    for (let i = 0; i < months.length; i++) {
        let month = months[i];
        let amount = totals[month];
        let width = amount / maximum * 100;

        html += `
            <div class="bar-item">
                <span>${escapeHTML(month)}</span>
                <div class="bar" style="width:${width}%"></div>
                <strong>${formatCurrency(amount)}</strong>
            </div>
        `;
    }

    chart.innerHTML = html || "<p>No monthly data available.</p>";
}


/* 9. REPORTS AND CSV EXPORT */

function updateReports() {
    let totalNet = 0;

    for (let i = 0; i < payrollRecords.length; i++) {
        totalNet += payrollRecords[i].net;
    }

    getElement("report-employees").textContent = employees.length;
    getElement("report-records").textContent = payrollRecords.length;
    getElement("report-net").textContent = formatCurrency(totalNet);
}

function downloadCSV(fileName, rows) {
    let csv = "";

    for (let i = 0; i < rows.length; i++) {
        let row = [];

        for (let j = 0; j < rows[i].length; j++) {
            let value = String(rows[i][j] ?? "").replace(/"/g, '""');
            row.push('"' + value + '"');
        }
        }
