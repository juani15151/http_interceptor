# http_interceptor

A middleware library that lets you modify requests and responses if desired. Based of on [http_middleware](https://github.com/TEDConsulting/http_middleware)

## Getting Started

This is a plugin that lets you intercept the different requests and responses from Dart's http package. You can use to add headers, modify query params, or print a log of the response.

### Installing

Include the package with the latest version available in your `pubspec.yaml`.

```dart
    http_interceptor: any
```

### Importing

```dart
import 'package:http_interceptor/http_interceptor.dart';
```

### Using `http_interceptor`

#### Building your own interceptor

In order to implement `http_interceptor` you need to implement the `InterceptorContract` and create your own interceptor. This abstract class has two methods: `interceptRequest`, which triggers before the http request is called; and `interceptResponse`, which triggers after the request is called, it has a response attached to it which the corresponding to said request. You could use this to do logging, adding headers, error handling, or many other cool stuff. It is important to note that after you proccess the request/response objects you need to return them so that `http` can continue the execute.

- Logging with interceptor:

```dart
class LogginInterceptor implements InterceptorContract {
  @override
  Future<RequestData> interceptRequest({RequestData data}) async {
    print(data);
    return data;
  }

  @override
  Future<ResponseData> interceptResponse({ResponseData data}) async {
      print(data);
      return data;
  }

}
```

- Changing headers with interceptor:

```dart
class WeatherApiInterceptor implements InterceptorContract {
  @override
  Future<RequestData> interceptRequest({RequestData data}) async {
    try {
      data.params['appid'] = OPEN_WEATHER_API_KEY;
      data.params['units'] = 'metric';
      data.headers["Content-Type"] = "application/json";
    } catch (e) {
      print(e);
    }
    return data;
  }

  @override
  Future<ResponseData> interceptResponse({ResponseData data}) async => data;
}
```

#### Using your interceptor

Now that you actually have your interceptor implemented, now you need to use it. There are two general ways in which you can use them: by using the `HttpWithInterceptor` to do separate connections for different requests or using a `HttpClientWithInterceptor` for keeping a connection alive while making the different `http` calls. The ideal place to use them is in the service/provider class or the repository class (if you are not using services or providers); if you don't know about the repository pattern you can just google it and you'll know what I'm talking about. ;)

##### Using interceptors with Client

Normally, this approach is taken because of its ability to be tested and mocked.

Here is an example with a repository using the `HttpClientWithInterceptor` class.

```dart
class WeatherRepository {
  Client client = HttpClientWithInterceptor.build(interceptors: [
      WeatherApiInterceptor(),
  ]);

  Future<Map<String, dynamic>> fetchCityWeather(int id) async {
    var parsedWeather;
    try {
      final response =
          await client.get("$baseUrl/weather", params: {'id': "$id"});
      if (response.statusCode == 200) {
        parsedWeather = json.decode(response.body);
      } else {
        throw Exception("Error while fetching. \n ${response.body}");
      }
    } catch (e) {
      print(e);
    }
    return parsedWeather;
  }

}
```

##### Using interceptors without Client

This is mostly the straight forward approach for a one-and-only call that you might need intercepted.

Here is an example with a repository using the `HttpWithInterceptor` class.

```dart
class WeatherRepository {


  Future<Map<String, dynamic>> fetchCityWeather(int id) async {
    var parsedWeather;
    try {
      var response = await HttpWithInterceptor.build(
              interceptors: [WeatherApiInterceptor()])
          .get("$baseUrl/weather", params: {'id': "$id"});
      if (response.statusCode == 200) {
        parsedWeather = json.decode(response.body);
      } else {
        throw Exception("Error while fetching. \n ${response.body}");
      }
    } catch (e) {
      print(e);
    }
    return parsedWeather;
  }

}
```

### Issue Reporting

Open an issue and tell me, I will be happy to help you out as soon as I can.
