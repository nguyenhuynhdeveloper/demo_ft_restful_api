26/10/2022
// https://pub.dev/packages/dio#interceptors

Một ứng dụng khách Http mạnh mẽ cho Dart, hỗ trợ Interceptors, cấu hình chung, dữ liệu biểu mẫu, hủy yêu cầu, tải xuống tệp, thời gian chờ, v.v.

Ví dụ siêu đơn giản : 
import 'package:dio/dio.dart';
void getHttp() async {
  try {
    var response = await Dio().get('http://www.google.com');
    print(response);
  } catch (e) {
    print(e);
  }
}


##### Dio APIs 
có thể tạo 1 phiên bản và đặt cấu hình mặc định cho DIO

//---Set cấu hinh mặc định chi DIO
var dio = Dio(); // with default Options

// Set default configs
dio.options.baseUrl = 'https://www.xx.com/api';
dio.options.connectTimeout = 5000; //5s
dio.options.receiveTimeout = 3000;


# Tạo 1 phiên bản DIO mới với cấu hình BaseOptions thay thế

var options = BaseOptions(
  baseUrl: 'https://www.xx.com/api',
  connectTimeout: 5000,
  receiveTimeout: 3000,
);
Dio dio = Dio(options);

Các phương thức Request với DIO
Future<Response> get(...)

Future<Response> post(...)

Future<Response> put(...)

Future<Response> delete(...)

Future<Response> head(...)

Future<Response> put(...)

Future<Response> path(...)

Future<Response> download(...)

Future<Response> fetch(RequestOptions)


# Request Options: class tuỳ chọn mô tả cấu hình và thông tin yêu cầu http , Mỗi đối tượng dio có 1 cấu hình cơ sở cho tất cả các yêu cầu 

{
  /// Http method.
  String method;

  /// Request base url, it can contain sub path, like: 'https://www.google.com/api/'.
  String baseUrl;

  /// Http request headers.
  Map<String, dynamic> headers;

   /// Timeout in milliseconds for opening  url.
  int connectTimeout;

   ///  Whenever more than [receiveTimeout] (in milliseconds) passes between two events from response stream,
  ///  [Dio] will throw the [DioError] with [DioErrorType.RECEIVE_TIMEOUT].
  ///  Note: This is not the receiving time limitation.
  int receiveTimeout;

  /// Request data, can be any type.
  T data;


  // nếu path bắt đầu với http(s) thì baseURL sẽ được bỏ qua, nếu không thì nó sẽ kết hợp và giải quyết với baseUrl
  String path='';



  // Content-Type : mặc định sẽ là 'application/json; charset=utf-8
  // và Dio sẽ tự động mã hoá request body : quyết định xem request/response truyền lên thuộc định dạng nào/ kết quả truyền xuống mặc định là kiểu nào
  String contentType;


  // Chỉ ra loại data mà server sẽ trả về bằng sự định nghĩa ResponseType là 'JSON' , "STREAM', 'PLAIN'
  Nếu contentType dạng 'application/json' thì response trả về default dạng JSON
  Nếu dạng 'STREAM'  thì response trả về dạng data với binary byte: ví dụ cho trường học dowload hình ảnh 
  Nếu dạng 'PLAIN' thì response trả về dạng String

  ResponseType responseType;


  validateStatus: định nghĩa khi nào request là thành công 
  ValidateStatus validateStatus;


  // tự custom cái mà sẽ nhận sau khi Interceptor và Transformer
  Map<String, dynamic> extra;
  

  // Định nghĩa tham số query chung
  Map<String, dynamic /*String|Iterable<String>*/ > queryParameters;  
  
 
    Định nghĩa format của thi thập data in request
  late CollectionFormat collectionFormat;

}


# Response Schema : Hình thái của của response
1 response cho 1 request chứa các nội dung như sau 
Khi 1 request thành công thì bạn sẽ nhận 1 response như định dạng 

{
  /// Response body. may have been transformed, please refer to [ResponseType].
  T? data;
  /// Response headers.
  Headers headers;
  /// The corresponding request info.
  Options request;
  /// Http status code.
  int? statusCode;
  String? statusMessage;
  /// Whether redirect 
  bool? isRedirect;  
  /// redirect info    
  List<RedirectInfo> redirects ;
  /// Returns the final real request uri (maybe redirect). 
  Uri realUri;    
  /// Custom field that you can retrieve it later in `then`.
  Map<String, dynamic> extra;
}


Response response = await dio.get('https://www.google.com');
print(response.data);
print(response.headers);
print(response.requestOptions);
print(response.statusCode);

# Interceptors : Máy đánh chặn mỗi đối tượng dio , chúnh ta có thể thêm một hoặc nhiều bộ chặn , để chặn các request, trước khi đưa chúng vào sử lý ở then hoặc catchError


dio.interceptors.add(InterceptorsWrapper(
    onRequest:(options, handler){
     // Do something before request is sent
     return handler.next(options); //continue
     // If you want to resolve the request with some custom data，
     // you can resolve a `Response` object eg: `handler.resolve(response)`.
     // If you want to reject the request with a error message,
     // you can reject a `DioError` object eg: `handler.reject(dioError)`
    },
    onResponse:(response,handler) {
     // Do something with response data
     return handler.next(response); // continue
     // If you want to reject the request with a error message,
     // you can reject a `DioError` object eg: `handler.reject(dioError)` 
    },
    onError: (DioError e, handler) {
     // Do something with response error
     return  handler.next(e);//continue
     // If you want to resolve the request with some custom data，
     // you can resolve a `Response` object eg: `handler.resolve(response)`.  
    }
));

Ví dụ 1 bộ chặn đơn giản : những hành động trước khi gửi đi request - trước khi nhận xuống response - trước khi nhận xuống lỗi : chỉ in ra 

import 'package:dio/dio.dart';
class CustomInterceptors extends Interceptor {
  @override
  void onRequest(RequestOptions options, RequestInterceptorHandler handler) {
    print('REQUEST[${options.method}] => PATH: ${options.path}');
    return super.onRequest(options, handler);
  }
  @override
  Future onResponse(Response response, ResponseInterceptorHandler handler) {
    print('RESPONSE[${response.statusCode}] => PATH: ${response.requestOptions.path}');
    return super.onResponse(response, handler);
  }
  @override
  Future onError(DioError err, ErrorInterceptorHandler handler) {
    print('ERROR[${err.response?.statusCode}] => PATH: ${err.requestOptions.path}');
    return super.onError(err, handler);
  }
}

# Resolve and reject the request
Trong tất cả các bộ chặn ,ta có thể can thiệp vào luồng thực thi của chúng , 
Nếu bạn muốn resolve the request/response với một số dữ liệu tùy chỉnh ， bạn có thể gọi handler.resolve (Response)
Nếu bạn muốn reject the request/response với error message , bạn có thể call handler.rejectt(dioError)

# Queued Interceptor : Xắp sếp bộ chặn 
Interceptor có thể được thực thi đồng thời, nghĩa là, tất cả các yêu cầu đều nhập vào interceptor cùng một lúc, thay vì thực hiện tuần tự,
Tuy nhiên trong một số trường hợp cần request nhập vào 1 các tuần tự , 
Do đó, chúng ta cần cung cấp một cơ chế để truy cập tuần tự （từng cái một） đến các bộ đánh chặn và QueuedInterceptor có thể giải quyết vấn đề này

Ví dụ : vì lý do bảo mật chính ta cần tất cả các request đều phải có csrfToken trong header , nếu không có  thì cần yêu cầu thêm 1 csrf
token vào trong header, sau đó mới gửi request đi 
Vì quá trình lấy csrfToken là không đồng bộ, nên chúng ta cần thực hiện yêu cầu này 1 cách không đồng bộ trong trình chặn yêu cầu 


  var dio = Dio();
  //  dio instance to request token
  var tokenDio = Dio();
  String? csrfToken;
  dio.options.baseUrl = 'http://www.dtworkroom.com/doris/1/2.0.0/';
  tokenDio.options = dio.options;
  dio.interceptors.add(QueuedInterceptorsWrapper(
    onRequest: (options, handler) {
      print('send request：path:${options.path}，baseURL:${options.baseUrl}');
      if (csrfToken == null) {
        print('no token，request token firstly...');
        tokenDio.get('/token').then((d) {
          options.headers['csrfToken'] = csrfToken = d.data['data']['token'];
          print('request token succeed, value: ' + d.data['data']['token']);
          print(
              'continue to perform request：path:${options.path}，baseURL:${options.path}');
          handler.next(options);
        }).catchError((error, stackTrace) {
          handler.reject(error, true);
        });
      } else {
        options.headers['csrfToken'] = csrfToken;
        return handler.next(options);
      }
    },
   ); 

   You can clean the waiting queue by calling clear();



# LogInterceptor
Log : in ra request / response 
Bạn có thể dùng LogInterceptor để có thể in request/response log ra 1 cách tự động 
EX: 
dio.interceptors.add(LogInterceptor(responseBody: false));

# Custom Interceptor
Bạn có thể tùy chỉnh thiết bị chặn bằng cách mở rộng lớp Interceptor/QueuedInterceptor

# Cookie Manager

##### Handling Errors
Khi có lỗi xảy ra , Dio sẽ gói Error/Exception to a DioError :

try {
  //404
  await dio.get('https://wendux.github.io/xsddddd');
} on DioError catch (e) {
  // The request was made and the server responded with a status code
  // that falls out of the range of 2xx and is also not 304.
  if (e.response != null) {
    print(e.response.data)
    print(e.response.headers)
    print(e.response.requestOptions)
  } else {
    // Something happened in setting up or sending the request that triggered an Error
    print(e.requestOptions)
    print(e.message)
  }
}




  # DioError scheme: hình thái của 1 DioError

 {
  /// Response info, it may be `null` if the request can't reach to
  /// the http server, for example, occurring a dns error, network is not available.
  Response? response;
  /// Request info.
  RequestOptions? request;
  /// Error descriptions.
  String message;

  DioErrorType type;
  /// The original error/exception object; It's usually not null when `type`
  /// is DioErrorType.DEFAULT
  // Nguyên bản error/exception , nó sẽ thường xuyên khác null khi type  is DioErrorType.DEFAULT
  dynamic? error;
}


# DioErrorType : Loại DioErrorType

enum DioErrorType {
  /// It occurs when url is opened timeout.
  connectTimeout,

  /// It occurs when url is sent timeout.
  sendTimeout,

  ///It occurs when receiving timeout.
  receiveTimeout,

  /// When the server response, but with a incorrect status, such as 404, 503...
  response,

  /// When the request is cancelled, dio will throw a error with this type.
  cancel,

  /// Default error type, Some other Error. In this case, you can
  /// use the DioError.error if it is not null.
  other,
}

#####  Using application/x-www-form-urlencoded format 
theo mặc định , Dio sẽ trả về response thành JSON, để có thể gửi data thành dạng application/x-www-form-urlencoded
bạn cần thay thế

//Instance level
dio.options.contentType= Headers.formUrlEncodedContentType;
//or works once
dio.post(
  '/info',
  data: {'id': 5},
  options: Options(contentType: Headers.formUrlEncodedContentType),
);


##### Sending FormData
Bạn có thể gửi formData với Dio , cái sẽ gửi data khi contentType dạng multipart/form-data , và nó sẽ hồ trợ upload file

var formData = FormData.fromMap({
  'name': 'wendux',
  'age': 25,
  'file': await MultipartFile.fromFile('./text.txt',filename: 'upload.txt')
});
response = await dio.post('/info', data: formData);

--Ví dụ trong case upload file image 

  void uploadImageRepresent(Map<String, dynamic> params, String path) async {
    Map<String, dynamic> body = {};
    body['files'] = await MultipartFile.fromFile(path, filename: 'image_represent.png');   // tham số 1: là đường dẫn , tham số 2: là namefile
    var formData = FormData.fromMap(body);     // convert hình ảnh sang dạng FormData
    try {
      var res = await getIt.get<Api>().uploadImageCustomerRepresent(formData);    // gửi formData lên backend
      final result = BaseApiResponse.fromJson(res); //Đây chính tạo 1 đối tượng từ namedConstructor
      if (result.success) {
      } else {
      }
        } 
    catch (e) {
    }
  }

# Multiple files upload 
   
   Có 2 cách để add multiple file to FormData, sự khác nhau duy nhất là là key của loại mảng

   FormData.fromMap({
  'files': [
    MultipartFile.fromFileSync('./example/upload.txt', filename: 'upload.txt'),
    MultipartFile.fromFileSync('./example/upload.txt', filename: 'upload.txt'),
  ]
});

# Reuse FormDatas and MultipartFiles 



   ----Ví dụ thực tế tại dự án Event----------
     static Dio provideDioAuth(SharedPreferenceHelper sharedPrefHelper) {
    final dio = Dio();

    dio
      ..options.baseUrl = dotenv.env['BASE_LOGIN_URL'] ?? ''
      ..options.connectTimeout = Urls.connectionTimeout
      ..options.receiveTimeout = Urls.receiveTimeout
      ..options.headers = {'Content-Type': 'application/json; charset=utf-8'}
      ..interceptors.add(LogInterceptor(        // Bước để có thể in ra request/ response 
        request: true,
        responseBody: true,
        requestBody: true,
        requestHeader: true,
      ))
      ..interceptors.add(
        InterceptorsWrapper(
          onRequest: (RequestOptions options,
              RequestInterceptorHandler handler) async {
            // getting token
            var token = await sharedPrefHelper.authToken;

            if (token != null) {
              options.headers.putIfAbsent('Authorization', () => 'Bearer ${token}');
            } else {
              print('Auth token is null================================>>>>>');
            }

            return handler.next(options);
          },
        ),
      );

      //check bad certificate
    (dio.httpClientAdapter as DefaultHttpClientAdapter).onHttpClientCreate = (HttpClient client) {
      client.badCertificateCallback = (X509Certificate cert, String host, int port) => true;
      return client;
    };

    return dio;
  }



//Ví dụ dự án 
class DioClient {
  // dio instance
  final Dio _dio;
  // injecting dio instance
  DioClient(this._dio);

  // Get:----------
  Future<dynamic> get(
      String uri, {
        Map<String, dynamic>? queryParameters,
        Options? options,
        CancelToken? cancelToken,
        ProgressCallback? onReceiveProgress,
      }) async {
    try {
      final Response response = await _dio.get(
        uri,
        queryParameters: queryParameters,
        options: options,
        cancelToken: cancelToken,
        onReceiveProgress: onReceiveProgress,
      );
      // return response.data;
      return BaseApiResponse(code: response.statusCode, data: response.data, message: response.statusMessage, success: true).toJson();
    } on DioError catch (error) { 
      return errorHandle(error: error);     //Khi mà có lỗi chỉ đơn giản return ra lỗi đó
    }
  }
  }