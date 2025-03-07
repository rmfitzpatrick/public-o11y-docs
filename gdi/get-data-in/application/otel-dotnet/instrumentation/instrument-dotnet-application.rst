.. _instrument-otel-dotnet-applications:

****************************************************************************
Instrument a .NET application for Splunk Observability Cloud (OpenTelemetry)
****************************************************************************

.. meta::
   :description: The Splunk Distribution of OpenTelemetry .NET automatically instruments .NET applications, Windows services running .NET applications, and ASP.NET applications deployed on IIS. Follow these steps to get started.

The Splunk Distribution of OpenTelemetry .NET automatically instruments .NET applications, Windows services running .NET applications, and ASP.NET applications deployed on IIS.

.. caution:: This distribution is currently in beta. Don't use it in production environments. Some features might have constrained capabilities. Support is provided on a best-effort basis.

To get started, use the guided setup or follow the instructions manually.

Generate customized instructions using the guided setup
====================================================================

To generate all the basic installation commands for your environment and application, use the .NET OpenTelemetry guided setup. To access the .NET OpenTelemetry guided setup, follow these steps:

#. Log in to Splunk Observability Cloud.
#. Open the :new-page:`.NET OpenTelemetry guided setup <https://login.signalfx.com/#/gdi/scripted/otel-dotnet-tracing/>`. Optionally, you can navigate to the guided setup on your own:

   #. In the navigation menu, select :menuselection:`Data Management`.
   #. Select :guilabel:`Add Integration` to open the :guilabel:`Integrate Your Data` page.
   #. In the integration filter menu, select :guilabel:`By Product`.
   #. Select the :guilabel:`APM` product.
   #. Select the :guilabel:`.NET (OpenTelemetry)` tile to open the .NET OpenTelemetry guided setup.

Install the Splunk Distribution of OpenTelemetry .NET manually
==================================================================

Follow these instructions to install the Splunk Distribution of OpenTelemetry .NET:

- :ref:`install-dotnet-otel-instrumentation`
- :ref:`configure-otel-dotnet`

To install the distribution using the official NuGet packages, see :ref:`otel-dotnet-nuget-pkg`.

.. _install-dotnet-otel-instrumentation:

Instrument a .NET application
---------------------------------------------

Follow these steps to automatically instrument your application:


Windows
^^^^^^^^^^^^

#. Check that you meet the requirements. See :ref:`dotnet-otel-requirements`.

#. (Optional) If needed, uninstall the SignalFx Instrumentation for .NET. See :ref:`uninstall-dotnet-sfx`.

#. Download and install the Splunk Distribution of OpenTelemetry .NET from the :new-page:`Releases page on GitHub <https://github.com/signalfx/splunk-otel-dotnet/releases>`. For example:

   .. code-block:: powershell

      # Download and import the PowerShell module
      # Replace <version> with the desired version
      $module_url = "https://github.com/signalfx/splunk-otel-dotnet/releases/download/<version>/Splunk.OTel.DotNet.psm1"
      $download_path = Join-Path $env:temp "Splunk.OTel.DotNet.psm1"
      Invoke-WebRequest -Uri $module_url -OutFile $download_path
      Import-Module $download_path

      # Install the Splunk distribution using the PowerShell module
      Install-OpenTelemetryCore

   .. note:: Replace ``<version>`` in the ``module_url`` with the desired version.

#. Register the distribution:

   .. tabs::

      .. code-tab:: shell .NET application

         # Set up environment to start instrumentation from the current PowerShell session
         Register-OpenTelemetryForCurrentSession -OTelServiceName "<your-service-name>"

      .. code-tab:: shell IIS application (.NET)

         # Set up IIS instrumentation
         # IIS is restarted as a result
         Register-OpenTelemetryForIIS

      .. code-tab:: shell Windows service

         # Set up your Windows Service instrumentation
         Register-OpenTelemetryForWindowsService -WindowsServiceName "<your-windows-service-name>"

#. Set the environment and service version resource attributes:

   .. code-block:: powershell

      # You can also set this in web.config or app.config
      $env:OTEL_RESOURCE_ATTRIBUTES='deployment.environment=<envtype>,service.version=<version>'

#. Run your application.

If no data appears in APM, see :ref:`common-dotnet-otel-troubleshooting`.

.. note:: If you need to add custom attributes to spans or want to manually generate spans and metrics, instrument your .NET application or service manually. See :ref:`dotnet-otel-manual-instrumentation`.

Linux
^^^^^^^^^^^^^^^^^

#. Check that you meet the requirements. See :ref:`dotnet-otel-requirements`.

#. (Optional) If needed, uninstall the SignalFx Instrumentation for .NET. See :ref:`uninstall-dotnet-sfx`.

#. Download and install the installation script of the Splunk Distribution of OpenTelemetry .NET from the :new-page:`Releases page on GitHub <https://github.com/signalfx/splunk-otel-dotnet/releases>`. For example:

   .. code-block:: shell

      # Replace <version> with the desired version
      curl -sSfL https://github.com/signalfx/splunk-otel-dotnet/releases/download/<version>/splunk-otel-dotnet-install.sh -O
      # Install the distribution
      sh ./splunk-otel-dotnet-install.sh

   .. note:: Replace ``<version>`` in the curl URL with the desired version.

#. Activate the automatic instrumentation:

   .. code-block:: shell

      # Activate the automatic instrumentation
      . $HOME/.splunk-otel-dotnet/instrument.sh

#. Set the environment and service version resource attributes:

   .. code-block:: shell

      export OTEL_RESOURCE_ATTRIBUTES='deployment.environment=<envtype>,service.version=<version>'

#. Run your application.

If no data appears in APM, see :ref:`common-dotnet-otel-troubleshooting`.

.. note:: If you need to add custom attributes to spans or want to manually generate spans, instrument your .NET application or service manually. See :ref:`dotnet-otel-manual-instrumentation`.

.. _configure-otel-dotnet:

Configure the instrumentation
---------------------------------------------

For advanced configuration of the .NET automatic instrumentation, like changing trace propagation formats or changing the endpoint URLs, see :ref:`advanced-dotnet-otel-configuration`.

.. _otel-dotnet-nuget-pkg:

Install the OpenTelemetry .NET instrumentation using the NuGet packages
--------------------------------------------------------------------------

You can deploy the Splunk Distribution of OpenTelemetry .NET instrumentation automatically through the official NuGet packages. The project of your instrumented application must support NuGet packages.

Use the NuGet package in the following scenarios:

1. You control the application build but not the machine or container where the application is running.
2. You're instrumenting a self-contained application. See :new-page:`Publish self-contained <https://learn.microsoft.com/en-us/dotnet/core/deploying/#publish-self-contained>` in the .NET documentation.
3. You want to facilitate developer experimentation with automatic instrumentation through NuGet packages.
4. You need to solve version conflicts between the dependencies used by the application and the automatic instrumentation.

Instrument an application using the NuGet packages
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To automatically instrument your application using the NuGet packages, add the ``Splunk.OpenTelemetry.AutoInstrumentation`` package to your project. For example:

.. code-block:: powershell

   dotnet add [<PROJECT>] package Splunk.OpenTelemetry.AutoInstrumentation --prerelease

If the build fails and prompts you to add missing instrumentation packages, add the instrumentation package or skip the instrumentation of the listed package by adding it to the ``SkippedInstrumentation`` property. For example:

.. code-block:: xml

   <PropertyGroup>
      <SkippedInstrumentations>MongoDB.Driver.Core;StackExchange.Redis</SkippedInstrumentations>
   </PropertyGroup>

You can also set the ``SkippedInstrumentation`` property from the terminal. Rewrite the ``;`` separator as ``%3B``. For example:

.. code-block:: powershell

   dotnet build -p:SkippedInstrumentations=StackExchange.Redis%3BMongoDB.Driver.Core

To distribute the appropriate native runtime components with your .NET application, specify a Runtime Identifier (RID) to build the application using ``dotnet build`` or ``dotnet publish``. 

Both self-contained and framework-dependent applications are compatible with automatic instrumentation. See :new-page:`.NET application publishing overview <https://learn.microsoft.com/en-us/dotnet/core/deploying/>` in the .NET documentation for more information.

Run the instrumented application
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Use the script in the output folder of the build to run the application with automatic instrumentation activated.

- On Windows, use ``splunk-launch.cmd <application_executable>``.
- On Linux, use ``splunk-launch.sh <application_executable>``.

If you run the application using the ``dotnet`` CLI, add ``dotnet`` after the script.

- On Windows, use ``splunk-launch.cmd dotnet <application>``.
- On Linux, use ``splunk-launch.sh dotnet <application>``.

The script passes all the command-line parameters you provide to the application.


.. _windows-offline-install-otel-dotnet:

Offline installation for Windows
----------------------------------------------

To install the .NET automatic instrumentation on Windows hosts that are offline, follow these steps:

#. Download the following files from the :new-page:`Releases page on GitHub <https://github.com/signalfx/splunk-otel-dotnet/releases>` and copy them to the offline server:

   - ``Splunk.OTel.DotNet.psm1``
   - ``splunk-opentelemetry-dotnet-windows.zip``

#. Import the PowerShell script manually by running the following command:

   .. code-block:: powershell

      # Make sure the Download path is correct

      Import-Module C:\Users\Administrator\Downloads\Splunk.OTel.DotNet.psm1

   When prompted, enter ``R`` for ``Run Once``.

#. Run the install command:

   .. code-block:: powershell

      # Make sure the Download path is correct

      Install-OpenTelemetryCore -LocalPath "C:\Users\Administrator\Downloads\splunk-opentelemetry-dotnet-windows.zip"

.. _export-directly-to-olly-cloud-dotnet-otel:

Send data directly to Splunk Observability Cloud
---------------------------------------------------

By default, all telemetry is sent to the local instance of the Splunk Distribution of OpenTelemetry Collector.

To bypass the OTel Collector and send data directly to Splunk Observability Cloud, set the following environment variables:

.. tabs::

   .. code-tab:: shell Windows PowerShell

      $env:SPLUNK_ACCESS_TOKEN=<access_token>
      $env:SPLUNK_REALM=<realm>

   .. code-tab:: shell Linux

      export SPLUNK_ACCESS_TOKEN=<access_token>
      export SPLUNK_REALM=<realm>

To obtain an access token, see :ref:`admin-api-access-tokens`.

In the ingest endpoint URL, ``realm`` is the Splunk Observability Cloud realm, for example, ``us0``. To find the realm name of your account, follow these steps:

#. Open the navigation menu in Splunk Observability Cloud.
#. Select :menuselection:`Settings`.
#. Select your username.

The realm name appears in the :guilabel:`Organizations` section.
