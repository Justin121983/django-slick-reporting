Django Slick Reporting
======================

A one stop reports engine with batteries included.

Features
--------

- Effortlessly create Simple, Grouped, Time series and Crosstab reports in a handful of code lines.
- Create Chart(s) for your reports with a single line of code.
- Create Custom complex Calculation.
- Optimized for speed.
- Easily extendable.

Installation
------------

Use the package manager `pip <https://pip.pypa.io/en/stable/>`_ to install django-slick-reporting.

.. code-block:: console

        pip install django-slick-reporting


Usage
-----

So we have a model `SalesTransaction` which contains typical data about a sale.
We can extract different kinds of information for that model.

Let's start by a "Group by" report. This will generate a report how much quantity and value was each product sold within a certain time.

.. code-block:: python


    # in views.py
    from django.db.models import Sum
    from slick_reporting.views import ReportView, Chart
    from slick_reporting.fields import ComputationField
    from .models import MySalesItems


    class TotalProductSales(ReportView):
        report_model = SalesTransaction
        date_field = "date"
        group_by = "product"
        columns = [
            "name",
            ComputationField.create(
                Sum, "quantity", verbose_name="Total quantity sold", is_summable=False
            ),
            ComputationField.create(
                Sum, "value", name="sum__value", verbose_name="Total Value sold $"
            ),
        ]

        chart_settings = [
            Chart(
                "Total sold $",
                Chart.BAR,
                data_source=["sum__value"],
                title_source=["name"],
            ),
            Chart(
                "Total sold $ [PIE]",
                Chart.PIE,
                data_source=["sum__value"],
                title_source=["name"],
            ),
        ]


    # then, in urls.py
    path("total-sales-report", TotalProductSales.as_view())

Time Series
-----------

A Time series report is a report that is generated for a periods of time.
The period can be daily, weekly, monthly, yearly or custom. Calculations will be performed for each period in the time series.

Example: How much was sold in value for each product monthly within a date period ?

.. code-block:: python

    # in views.py
    from slick_reporting.views import ReportView
    from slick_reporting.fields import ComputationField
    from .models import SalesTransaction


    class MonthlyProductSales(ReportView):
        report_model = SalesTransaction
        date_field = "date"
        group_by = "product"
        columns = ["name", "sku"]

        time_series_pattern = "monthly"
        # or "yearly" , "weekly" , "daily" , others and custom patterns
        time_series_columns = [
            ComputationField.create(
                Sum, "value", verbose_name=_("Sales Value"), name="value"
            )  # what will be calculated for each month
        ]

        chart_settings = [
            Chart(
                _("Total Sales Monthly"),
                Chart.PIE,
                data_source=["value"],
                title_source=["name"],
                plot_total=True,
            ),
            Chart(
                "Total Sales [Area chart]",
                Chart.AREA,
                data_source=["value"],
                title_source=["name"],
                plot_total=False,
            ),
        ]

Cross Tab
---------
Use crosstab reports, also known as matrix reports, to show the relationships between three or more query items.
Crosstab reports show data in rows and columns with information summarized at the intersection points.

.. code-block:: python

        # in views.py
        from slick_reporting.views import ReportView
        from slick_reporting.fields import ComputationField
        from .models import MySalesItems


        class MyCrosstabReport(ReportView):

            crosstab_field = "client"
            crosstab_ids = [1, 2, 3]
            crosstab_columns = [
                ComputationField.create(Sum, "value", verbose_name=_("Value for")),
            ]
            crosstab_compute_remainder = True

            columns = [
                "some_optional_field",
                # You can customize where the crosstab columns are displayed in relation to the other columns
                "__crosstab__",
                # This is the same as the Same as the calculation in the crosstab, but this one will be on the whole set. IE total value
                ComputationField.create(Sum, "value", verbose_name=_("Total Value")),
            ]

Low level
---------

The view is a wrapper over the `ReportGenerator` class, which is the core of the reporting engine.
You can interact with the `ReportGenerator` using same syntax as used with the `ReportView` .

.. code-block:: python

    from slick_reporting.generator import ReportGenerator
    from .models import MySalesModel


    class MyReport(ReportGenerator):
        report_model = MySalesModel
        group_by = "product"
        columns = ["title", "__total__"]


    # OR
    my_report = ReportGenerator(
        report_model=MySalesModel, group_by="product", columns=["title", "__total__"]
    )
    my_report.get_report_data()  # -> [{'title':'Product 1', '__total__: 56}, {'title':'Product 2', '__total__: 43}, ]


This is just a scratch of what you can do and customize.

Road Ahead
----------

* Continue on enriching the demo project
* Add the dashboard capabilities
