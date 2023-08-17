# Custom configuration for application actuator endpoints

This topic for developers tells you how to configure the Application Live View connector component
to access actuator endpoints for custom settings, such as a different base path.
By default, the actuator endpoint for an application is exposed on `/actuator`.

The following table describes the actuator configuration scenarios and the
associated labels to use, assuming that the app runs on port `8080`:

| management.server.base-path | management.server.port | management.endpoints.web.base-path | server.servlet.context.path | Comments | Connector Configuration
| --- | --- | --- | --- | --- | --- | --- |
| None | None | None | None | Actuators endpoints available at `localhost:8080/actuator` | `tanzu.app.live.view.application.actuator.path=actuator`, `tanzu.app.live.view.application.actuator.port=8080`
| `/path` | `8082` | `/` | None |  Actuator endpoints available at `localhost:8082/path` | `tanzu.app.live.view.application.actuator.path=path`, `tanzu.app.live.view.application.actuator.port=8082`
| `/path` | `8082` | `/manage/actuator` | None |  Actuator endpoints available at `localhost:8082/path/manage/actuator` | `tanzu.app.live.view.application.actuator.path=path/manage/actuator`, `tanzu.app.live.view.application.actuator.port=8082`
| None | None | `/` | None | Actuators are deactivated to avoid conflicts | None
| None | None | `/manage` | None | Actuator endpoints available at `/manage` | `tanzu.app.live.view.application.actuator.path=manage`, `tanzu.app.live.view.application.actuator.port=8080`
| `/path` | `8082` | None | None | Actuator endpoints available at `localhost:8082/path/actuator` | `tanzu.app.live.view.application.actuator.path=path/actuator`, `tanzu.app.live.view.application.actuator.port=8082`
| `/` | `8082` | None | None | Actuator endpoints available at `localhost:8082/actuator` | `tanzu.app.live.view.application.actuator.path=actuator`, `tanzu.app.live.view.application.actuator.port=8082`
| None | None | None | `/api` | Actuator endpoints available at `localhost:8080/api/actuator` | `tanzu.app.live.view.application.actuator.path=api/actuator`, `tanzu.app.live.view.application.actuator.port=8080`
| `/path` | `8082` | None | `/api `| Actuator endpoints available at `localhost:8082/path/actuator` | `tanzu.app.live.view.application.actuator.path=path/actuator`, `tanzu.app.live.view.application.actuator.port=8082`
| `/path` | `8082` | `/manage` | `/api` | Actuator endpoints available at `localhost:8082/path/manage` | `tanzu.app.live.view.application.actuator.path=path/manage`, `tanzu.app.live.view.application.actuator.port=8082`
| `/path` | None | `/manage` | `/api` | Actuator endpoints available at `localhost:8080/api/manage` | `tanzu.app.live.view.application.actuator.path=api/manage`, `tanzu.app.live.view.application.actuator.port=8080`
| `/path` | None | `/` | `/api` | Actuators are deactivated to avoid conflicts | None
| `/path` | `8082` | `/` | `/api` | Actuator endpoints available at `localhost:8082/path` | `tanzu.app.live.view.application.actuator.path=path`, `tanzu.app.live.view.application.actuator.port=8082`
| None | None | `/manage` | `/api` | Actuator endpoints available at `localhost:8080/api/manage` | `tanzu.app.live.view.application.actuator.path=api/manage`, `tanzu.app.live.view.application.actuator.port=8080`