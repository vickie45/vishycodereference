Here's the modified code to generate bills every month automatically:

```python
from pathlib import Path
from datetime import datetime, timedelta
import calendar

from reportlab.lib.pagesizes import A4
from reportlab.lib.styles import getSampleStyleSheet, ParagraphStyle
from reportlab.lib.enums import TA_CENTER, TA_RIGHT
from reportlab.platypus import (
    SimpleDocTemplate,
    Paragraph,
    Spacer,
    Table,
    TableStyle
)
from reportlab.lib import colors
from reportlab.lib.units import mm


# ============================================================
# CONFIGURATION
# ============================================================

EMPLOYEE_NAME = "Vignesh"
TAX_REGIME = "New Tax Regime"

# Set this to True to generate bills for the current month
# Set to False and specify START_DATE and END_DATE for a range
GENERATE_CURRENT_MONTH = True

# If GENERATE_CURRENT_MONTH is False, set the date range here
# Format: "YYYY-MM-DD"
START_DATE = "2026-01-01"
END_DATE = "2026-12-31"

MONTHLY_CLAIMS = {
    "Conveyance": 7000,
    "Restaurant": 8800,
    "Telephone_Internet_Mobile": 980,
    "Furniture": 820,
}

OUTPUT_DIR = Path("monthly_claims")


# ============================================================
# HELPERS
# ============================================================

def get_months_to_generate():
    """Returns a list of (year, month) tuples for which to generate bills"""
    
    if GENERATE_CURRENT_MONTH:
        # Generate only for current month
        now = datetime.now()
        return [(now.year, now.month)]
    
    else:
        # Generate for the specified date range
        start = datetime.strptime(START_DATE, "%Y-%m-%d")
        end = datetime.strptime(END_DATE, "%Y-%m-%d")
        
        months = []
        current = start.replace(day=1)
        
        while current <= end:
            months.append((current.year, current.month))
            # Move to next month
            if current.month == 12:
                current = current.replace(year=current.year + 1, month=1)
            else:
                current = current.replace(month=current.month + 1)
        
        return months


def get_claim_date(year, month):
    """Returns a datetime object for the given year and month"""
    return datetime(year, month, 1)


def money(amount):
    return f"Rs. {amount:,.2f}"


def safe_filename(name):
    return (
        name.replace("/", "_")
            .replace("\\", "_")
            .replace(" ", "_")
    )


def create_styles():

    styles = getSampleStyleSheet()

    styles.add(
        ParagraphStyle(
            name="ClaimTitle",
            parent=styles["Title"],
            alignment=TA_CENTER,
            fontSize=18,
            spaceAfter=15,
        )
    )

    styles.add(
        ParagraphStyle(
            name="RightAligned",
            parent=styles["Normal"],
            alignment=TA_RIGHT,
        )
    )

    return styles


# ============================================================
# CREATE INDIVIDUAL CLAIM
# ============================================================

def create_claim_pdf(
    category,
    amount,
    claim_date,
    output_file
):

    styles = create_styles()

    document = SimpleDocTemplate(
        str(output_file),
        pagesize=A4,
        rightMargin=20 * mm,
        leftMargin=20 * mm,
        topMargin=18 * mm,
        bottomMargin=18 * mm,
    )

    content = []

    # --------------------------------------------------------
    # TITLE
    # --------------------------------------------------------

    content.append(
        Paragraph(
            "EMPLOYEE EXPENSE CLAIM",
            styles["ClaimTitle"]
        )
    )

    content.append(
        Paragraph(
            f"<b>Expense Category:</b> {category}",
            styles["Normal"]
        )
    )

    content.append(Spacer(1, 10))

    # --------------------------------------------------------
    # EMPLOYEE INFORMATION
    # --------------------------------------------------------

    employee_information = [
        ["Employee", EMPLOYEE_NAME],
        ["Tax Regime", TAX_REGIME],
        [
            "Claim Month",
            claim_date.strftime("%B %Y")
        ],
        [
            "Claim Date",
            claim_date.strftime("%d-%b-%Y")
        ],
    ]

    employee_table = Table(
        employee_information,
        colWidths=[45 * mm, 110 * mm]
    )

    employee_table.setStyle(
        TableStyle([
            (
                "GRID",
                (0, 0),
                (-1, -1),
                0.5,
                colors.grey
            ),
            (
                "BACKGROUND",
                (0, 0),
                (0, -1),
                colors.lightgrey
            ),
            (
                "FONTNAME",
                (0, 0),
                (0, -1),
                "Helvetica-Bold"
            ),
            (
                "VALIGN",
                (0, 0),
                (-1, -1),
                "MIDDLE"
            ),
            (
                "PADDING",
                (0, 0),
                (-1, -1),
                7
            ),
        ])
    )

    content.append(employee_table)

    content.append(Spacer(1, 20))

    # --------------------------------------------------------
    # CLAIM AMOUNT
    # --------------------------------------------------------

    claim_table_data = [
        ["Description", "Eligible Amount"],
        [category, money(amount)],
        ["TOTAL CLAIM", money(amount)],
    ]

    claim_table = Table(
        claim_table_data,
        colWidths=[110 * mm, 45 * mm]
    )

    claim_table.setStyle(
        TableStyle([
            (
                "GRID",
                (0, 0),
                (-1, -1),
                0.5,
                colors.grey
            ),
            (
                "BACKGROUND",
                (0, 0),
                (-1, 0),
                colors.lightgrey
            ),
            (
                "FONTNAME",
                (0, 0),
                (-1, 0),
                "Helvetica-Bold"
            ),
            (
                "FONTNAME",
                (0, -1),
                (-1, -1),
                "Helvetica-Bold"
            ),
            (
                "ALIGN",
                (1, 1),
                (1, -1),
                "RIGHT"
            ),
            (
                "PADDING",
                (0, 0),
                (-1, -1),
                8
            ),
        ])
    )

    content.append(claim_table)

    content.append(Spacer(1, 25))

    # --------------------------------------------------------
    # DECLARATION
    # --------------------------------------------------------

    content.append(
        Paragraph(
            "<b>Declaration</b>",
            styles["Heading3"]
        )
    )

    content.append(
        Paragraph(
            "I confirm that this claim is submitted in accordance "
            "with the applicable company reimbursement/allowance "
            "policy and represents my eligible monthly claim.",
            styles["Normal"]
        )
    )

    content.append(Spacer(1, 35))

    # --------------------------------------------------------
    # SIGNATURE
    # --------------------------------------------------------

    signature_table = Table(
        [
            ["Employee Signature", "Date"],
            [
                "________________________",
                "________________"
            ],
        ],
        colWidths=[90 * mm, 65 * mm]
    )

    signature_table.setStyle(
        TableStyle([
            (
                "FONTNAME",
                (0, 0),
                (-1, 0),
                "Helvetica-Bold"
            ),
            (
                "PADDING",
                (0, 0),
                (-1, -1),
                5
            ),
        ])
    )

    content.append(signature_table)

    document.build(content)


# ============================================================
# CREATE MONTHLY SUMMARY
# ============================================================

def create_summary_pdf(
    claim_date,
    output_file
):

    styles = create_styles()

    total = sum(MONTHLY_CLAIMS.values())

    document = SimpleDocTemplate(
        str(output_file),
        pagesize=A4,
        rightMargin=20 * mm,
        leftMargin=20 * mm,
        topMargin=18 * mm,
        bottomMargin=18 * mm,
    )

    content = []

    content.append(
        Paragraph(
            "MONTHLY CLAIM SUMMARY",
            styles["ClaimTitle"]
        )
    )

    content.append(
        Paragraph(
            f"<b>Employee:</b> {EMPLOYEE_NAME}<br/>"
            f"<b>Tax Regime:</b> {TAX_REGIME}<br/>"
            f"<b>Claim Month:</b> "
            f"{claim_date.strftime('%B %Y')}",
            styles["Normal"]
        )
    )

    content.append(Spacer(1, 20))

    # --------------------------------------------------------
    # SUMMARY TABLE
    # --------------------------------------------------------

    rows = [
        ["Category", "Monthly Eligible Amount"]
    ]

    for category, amount in MONTHLY_CLAIMS.items():

        rows.append([
            category,
            money(amount)
        ])

    rows.append([
        "TOTAL",
        money(total)
    ])

    summary_table = Table(
        rows,
        colWidths=[110 * mm, 45 * mm]
    )

    summary_table.setStyle(
        TableStyle([
            (
                "GRID",
                (0, 0),
                (-1, -1),
                0.5,
                colors.grey
            ),
            (
                "BACKGROUND",
                (0, 0),
                (-1, 0),
                colors.lightgrey
            ),
            (
                "FONTNAME",
                (0, 0),
                (-1, 0),
                "Helvetica-Bold"
            ),
            (
                "FONTNAME",
                (0, -1),
                (-1, -1),
                "Helvetica-Bold"
            ),
            (
                "ALIGN",
                (1, 1),
                (1, -1),
                "RIGHT"
            ),
            (
                "PADDING",
                (0, 0),
                (-1, -1),
                8
            ),
        ])
    )

    content.append(summary_table)

    content.append(Spacer(1, 25))

    content.append(
        Paragraph(
            f"<b>Total Monthly Eligible Claim: "
            f"{money(total)}</b>",
            styles["Heading3"]
        )
    )

    content.append(Spacer(1, 20))

    content.append(
        Paragraph(
            "This summary is intended for submission under "
            "the employer's applicable monthly claim/allowance "
            "process.",
            styles["Normal"]
        )
    )

    document.build(content)


# ============================================================
# MAIN PROGRAM
# ============================================================

def main():

    # Get the list of months to generate
    months_to_generate = get_months_to_generate()
    
    print()
    print("=" * 60)
    print("MONTHLY CLAIM GENERATOR")
    print("=" * 60)
    print(f"Employee : {EMPLOYEE_NAME}")
    print(f"Generating for {len(months_to_generate)} month(s)")
    print()
    
    total_overall = 0
    month_count = 0
    
    for year, month in months_to_generate:
        month_count += 1
        claim_date = get_claim_date(year, month)
        month_name = claim_date.strftime("%B %Y")
        
        print("-" * 60)
        print(f"Month {month_count}: {month_name}")
        print("-" * 60)
        
        # Create month folder
        month_folder = (
            OUTPUT_DIR /
            claim_date.strftime("%Y-%m")
        )
        
        month_folder.mkdir(
            parents=True,
            exist_ok=True
        )
        
        # ----------------------------------------------------
        # GENERATE CLAIMS FOR THIS MONTH
        # ----------------------------------------------------
        
        monthly_total = 0
        
        for category, amount in MONTHLY_CLAIMS.items():
            
            filename = (
                safe_filename(category)
                + "_Claim.pdf"
            )
            
            output_file = (
                month_folder /
                filename
            )
            
            create_claim_pdf(
                category=category,
                amount=amount,
                claim_date=claim_date,
                output_file=output_file
            )
            
            print(f"  Created: {filename}")
            monthly_total += amount
        
        # ----------------------------------------------------
        # GENERATE SUMMARY FOR THIS MONTH
        # ----------------------------------------------------
        
        summary_file = (
            month_folder /
            "Monthly_Claim_Summary.pdf"
        )
        
        create_summary_pdf(
            claim_date=claim_date,
            output_file=summary_file
        )
        
        print(f"  Created: Monthly_Claim_Summary.pdf")
        print(f"  Monthly Total: {money(monthly_total)}")
        print(f"  Saved in: {month_folder}")
        print()
        
        total_overall += monthly_total
    
    # Print overall summary
    print("=" * 60)
    print("GENERATION COMPLETE")
    print("=" * 60)
    print(f"Total months processed: {month_count}")
    print(f"Grand Total: {money(total_overall)}")
    print(f"Output directory: {OUTPUT_DIR.resolve()}")
    print()


if __name__ == "__main__":
    main()
```

Key Changes Made:

1. Added Month Generation Logic:
   · GENERATE_CURRENT_MONTH: Set to True for current month only, or False for a date range
   · START_DATE and END_DATE: Used when generating for a range of months
   · get_months_to_generate(): Returns a list of (year, month) tuples
2. Modified main() Function:
   · Loops through all months to generate
   · Creates separate folders for each month (e.g., 2026-01, 2026-02, etc.)
   · Generates all claim PDFs and summary for each month
   · Shows progress for each month being processed
   · Displays overall summary at the end
3. Usage Options:
   For Current Month Only:
   ```python
   GENERATE_CURRENT_MONTH = True
   # START_DATE and END_DATE are ignored
   ```
   For Date Range (e.g., January to December 2026):
   ```python
   GENERATE_CURRENT_MONTH = False
   START_DATE = "2026-01-01"
   END_DATE = "2026-12-31"
   ```
4. Output Structure:
   ```
   monthly_claims/
   ├── 2026-01/
   │   ├── Conveyance_Claim.pdf
   │   ├── Furniture_Claim.pdf
   │   ├── Restaurant_Claim.pdf
   │   ├── Telephone_Internet_Mobile_Claim.pdf
   │   └── Monthly_Claim_Summary.pdf
   ├── 2026-02/
   │   └── ... (same structure)
   └── ...
   ```

The script will now automatically generate bills for all specified months with the same amounts and formatting.