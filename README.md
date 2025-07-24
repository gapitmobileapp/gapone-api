# gapone-api

```yaml
openapi: 3.0.0
info:
  title: GapOne API
  x-logo:
    - url: "https://app.gapone.vn/content/production/images/gapit-icon.png"
  version: "1.6"
  description: |
    # Mục đích tài liệu
    Tài liệu này mô tả các thông tin kỹ thuật hỗ trợ cho việc thiết lập và vận  hành kênh trao đổi số liệu tin nhắn đa kênh (multi-channel A2P message) giữa đối tác và hệ thống GAPONE.

    Tải file [Postman Collection](https://docs.gapone.vn/gapit_api.postman_collection.json) để thử nghiệm kết nối.
    # Lịch sử thay đổi

    ### v1.6
      - [Whatsapp](#tag/Whatsapp_Channel): Cập nhật gửi tin whatsapp theo template
  
    ### v1.5
      - [Zalo](#tag/Zalo_Channel): Hỗ trợ gửi ZNS bằng hash phone
      - [Zalo_RSA](#tag/Zalo_RSA_Channel): Hỗ trợ mã hóa phone number
      - [Delivery](#tag/dn_overview): Bổ xung thông tin delivery notification
  
    ### v1.4
      - [Viber](#tag/Viber_Channel): Hỗ trợ thêm các loại tin Viber: TEXT, TEXT_TEMPLATE, IMG, FILE, VIDEO, PREDEFINED_TEMPLATE
      
    ### v1.3
      - [Basic Auth](#section/Authentication/BasicAuth): Hỗ trợ phương thức xác thực [Basic Authentication](https://developer.mozilla.org/en-US/docs/Web/HTTP/Authentication)

    ### v1.2
      - [Zalo API](#tag/Zalo):  Api liệt kê danh sách OA, ZNS template, tạo RSA key, tạo token tin hành trình
      - [Brandname API](#tag/Brandname): API liệt kê danh sách các Brandname(Sender) theo các kênh SMS, Viber, Whatsapp''

    ### v1.1 
      - [Tracking code](#tag/Tracking_Introduction)
      - [Tracking Event API](#tag/Tracking_Event_Server) 

    ### v1.0
      - [A2P Message API](#tag/A2P-Message)
      - [Contact API](#tag/Contact)
      - [Deal API](#tag/Deal)
      - [Product API](#tag/Product)
    # Quy trình kết nối dịch vụ
    Người dùng đăng nhập website GAPONE tại [https://app.gapone.vn](https://app.gapone.vn/OAuth2Application) để:
    ## Tạo API keys (client_id và client_secret):
    Các bước thực hiện:
      - Bước 1, 2, 3 (Step 1, 2, 3): Cài đặt  >  Kết nối API  >  Tạo mới
      - Bước 4 (Step 4): Điền tên ứng dụng.
      - Bước 5 (Step 5): Nhấn để xác nhận đăng ký ứng dụng. Ứng dụng mới với client_id và client_secret sẽ xuất hiện trong bảng.
       ![img1](https://docs.gapone.vn/vi/api_key.png)
     
    ## Đăng ký URL (webhook) nhận delivery notification:
    *Xem thêm về [nhận thông báo trạng thái](#tag/dn_overview)*.

    Các bước thực hiện:
     * Bước 1 (Step 1): Nhấn vào tên ứng dụng.
        ![img2](https://docs.gapone.vn/vi/webhook.png)
     * Bước 2 (Step 2): Điền Webhook Url.
     * Bước 3 (Step 3): Nhấn để xác nhận đăng ký Webhook Url.
       ![img3](https://docs.gapone.vn/vi/webhook2.png)
       Thông tin trên được sử dụng để thiết lập hệ thống client theo **[Thủ tục kết nối api](#section/Thu-tuc-ket-noi-api)**.
       Sau khi cấu hình đã đăng ký được kích hoạt, hệ thống có thể bắt đầu thực hiện việc gửi tin nhắn
       (A2P message) và nhận thông báo trạng thái [(delivery notification)](#tag/dn_overview).
    # Thủ tục kết nối api

    Gapone hỗ trợ 2 phương thức xác thực khi kết nối API:

    ### Basic Authentication:
      Xem [tại đây](#section/Authentication/BasicAuth)

    ### OAuth 2.0:
      Kết nối Gateway-API sử dụng cơ chế bảo mật dựa trên OAuth 2.0 Authorization Framework với grant-type là **[Client Credential Grant](https://tools.ietf.org/html/rfc6749#section-4.4)**.

      Với flow này, khi gửi A2P-message vào Gateway-API server (resource server), client request cần gửi kèm ACCESS_TOKEN hợp lệ (do Auth-API server cấp) theo cơ chế xác thực **[HTTP Bearer authentication cheme](#section/Authentication/OAuth2ClientCredsAuth)**.

      ![Flow](https://docs.gapone.vn/vi/flow.png)
  contact:
    email: info@gapone.vn
servers:
  - url: "https://gateway.gapone.vn"
    description: API
paths:
  /a2p-request:
    post:
      summary: Gửi tin nhắn A2P-message đến Gateway-API server
      operationId: post-v1-message
      responses:
        "200":
          description: OK (Click to see RESPONSE SCHEMA)
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/SentResponse"
              examples:
                example-1:
                  value:
                    message_id: 4172d060-937d-11ea-8dd3-83fb5ae4793c
                    request_id: 1312rfg3dgdgd
        "400":
          description: Bad Request. Kiểm tra tham số request. Không retry.
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/ErrorResponse"
              examples:
                example-1:
                  value:
                    request_id: 1312rfg3dgdgd
                    error: Invalid channel
        "401":
          description: Unauthorized. Kiểm tra tham số authentication (client_id & client_secret). Không retry.
        "402":
          description: Payment Required. Tạm ngừng gửi tin. Kiểm tra tài khoản (đối với trả trước) hoặc liên hệ Customer Support.
        "403":
          description: Forbidden. Kiểm tra tham số đường dẫn và quyền truy cập. Không retry.
        "422":
          description: Unprocessable. Nội dung request không hợp lệ. Kiểm tra lại tham số tin nhắn. Không retry.
        "429":
          description: Too many requests. Tốc độ gửi request quá cao. Cần giảm tốc độ gửi requests xuống mức cho phép (default 100 requests/second).
        "503":
          description: Server busy. Thử lại sau 5-10 giây.
        4xx:
          description: Bad Request. Kiểm tra tham số request. Không retry.
        5xx:
          description: Lỗi server. Thử lại sau 5-10 giây. Liên hệ Customer Support.
      description: |
        Gateway-API trao đổi tham số qua HTTP request/response headers & body như bên dưới.

        Cơ chế xác thực sử dụng HTTP Bearer auhentication scheme, với token là giá trị access_token
        nhận được từ Auth-API server ở [bước trên](#section/Authentication/OAuth2ClientCredsAuth) .

        Trước khi gửi tin phải đăng ký Brandname(Sender). Xem danh dánh Brandname(Sender) [ZNS,Zalo tại đây](#tag/Zalo/paths/~1v1~1zns~1oas/get), các kênh [SMS,Viber,Whatsapp tại đây](#tag/Brandname)

        Người dùng phải request để [Lấy trạng thái gửi tin](#operation/post-status-query) hoặc cài đặt [webhook](#section/Quy-trinh-ket-noi-dich-vu/DJang-ky-URL-(webhook)-nhan-delivery-notification:) để nhận [thông báo trạng thái gửi tin](#tag/dn_overview)

        Tải file [Postman Collection](https://docs.gapone.vn/gapit_api.postman_collection.json) để thử nghiệm kết nối.


        <span style="color:red"> Chú ý: request_id phải là duy nhất trong vòng 24h. Nếu gửi trùng request_id sẽ nhận lại được message_id của bản tin có request_id đầu tiên và việc gửi tin sẽ không được thực hiện</span>.
      security:
        - OAuth2ClientCredsAuth: []
        - BasicAuth: []
      x-codeSamples:
        - lang: JavaScript
          source: |
            var axios = require('axios');
            var data = JSON.stringify({"request_id":"4242342","message":{"channel":"zalo","from":"2293893445652123","to":"84912345678","template_id":"1234","template_data":{"OTP":123456,"customer_name":"Mr.HUY"}}});

            var config = {
              method: 'post',
              url: 'https://gateway.gapone.vn/a2p-request',
              headers: { 
                'Authorization': 'Bearer {ACCESS_TOKEN}', 
                'Content-Type': 'application/json'
              },
              data : data
            };

            axios(config)
            .then(function (response) {
              console.log(JSON.stringify(response.data));
            })
            .catch(function (error) {
              console.log(error);
            });
        - lang: curl
          source: |
            curl --location --request POST 'https://gateway.gapone.vn/a2p-request' \
            --header 'Authorization: Bearer {ACCESS_TOKEN}' \
            --header 'Content-Type: application/json' \
            --data-raw '{

                "request_id": "4242342",
                "message": 
            {
                 "channel": "zalo",
                "from": "2293893445652123",
                "to": "84912345678",
                 "template_id": "1234",
                "template_data": {"OTP":123456,"customer_name":"Mr.HUY"}
            }
            }'
        - lang: C#
          source: |
            //send message with sms channel
            var body = new
            {
                request_id = Guid.NewGuid().ToString(),
                message = new
                {
                    request_id = Guid.NewGuid().ToString(), // unique request id, required
                    channel = "sms", // required
                    from = "BrandName", //sms sender, required
                    to = "84332584400", //recipient number, required
                    content = new // sms content, required
                    {
                        text = "Hello",// required
                        msg_type = "text", // sms content type, required
                        sms_type = "CSKH", // sms type, required
                    }
                }
            };
            string Jbody = JsonConvert.SerializeObject(body);
            string url = "https://gateway.gapone.vn/a2p-request";
            using var client = new HttpClient();
            client.DefaultRequestHeaders.Clear();
            client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", token);
            client.DefaultRequestHeaders.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
            var data = new StringContent(Jbody, Encoding.UTF8, "application/json");
            HttpResponseMessage response = await client.PostAsync(url, data);
            response.EnsureSuccessStatusCode();
            var body = await response.Content.ReadAsStringAsync();
            Console.WriteLine(body);

            //send message with zalo channel
            var body = new
            {
                request_id = Guid.NewGuid().ToString(),
                message = new
                {
                    channel = "zalo", // required
                    from = "2293893412345665977", //zalo official account ID, required
                    to = "84332584400", //zalo recipient phone number, required

                    // template_id and template_data are provied after subscription 
                    template_id = "298759", //zalo template id, required
                    template_data = new //template custom data, required
                    {
                        customer_name = "Name",
                        OTP = "1234"
                    }
                }
            };
            string Jbody = JsonConvert.SerializeObject(body);
            string url = "https://gateway.gapone.vn/a2p-request";
            using var client = new HttpClient();
            client.DefaultRequestHeaders.Clear();
            client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", token);
            client.DefaultRequestHeaders.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
            var data = new StringContent(Jbody, Encoding.UTF8, "application/json");
            HttpResponseMessage response = await client.PostAsync(url, data);
            response.EnsureSuccessStatusCode();
            var body = await response.Content.ReadAsStringAsync();
            Console.WriteLine(body);

            //send message with viber channel
            var body = new
            {
                request_id = Guid.NewGuid().ToString(),
                message = new
                {
                    channel = "viber", // required
                    from = new
                    {
                        name = "BrandName", // brandname, required
                    },
                    to = "84332584400", //  viber recipient number, required
                    content = new
                    {
                        text = "Nội dung tin nhắn",// viber text content, required
                        media = new // viber media content, required
                        {
                            type = "image/jpeg",// media content type, required
                            name = "image",// media name
                            url = "https://icdn.dantri.com.vn/zoom/528_352/2021/10/10/covidhanoi-1633821326759.jpg", // media resource url, required   
                        }  ,
                        action = new // viber button action
                        {
                            type = "url",// action type, required
                            text = "Đăng ký ngay",// button caption, required
                            url = "https://my.vnpt.com.vn/ads?a=rRwec324c7",// action url, required
                        }
                    }
                }
            };
            string Jbody = JsonConvert.SerializeObject(body);
            string url = "https://gateway.gapone.vn/a2p-request";
            using var client = new HttpClient();
            client.DefaultRequestHeaders.Clear();
            client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", token);
            client.DefaultRequestHeaders.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
            var data = new StringContent(Jbody, Encoding.UTF8, "application/json");
            HttpResponseMessage response = await client.PostAsync(url, data);
            response.EnsureSuccessStatusCode();
            var body = await response.Content.ReadAsStringAsync();
            Console.WriteLine(body);

            // send message with email channel
            string[] mail = {"nch2699@gmail.com"};
            var body = new
            {
                request_id = Guid.NewGuid().ToString(),
                message = new
                {
                    channel = "email", // required
                    from = "name@gmail.vn", // email sender, required
                    to = mail, // recipient, required
                    subject = "thong bao tai khoan",// email subject, required
                    content = new // email body
                    {
                        text = "Thong bao tai khoan",
                        html = "<h5>Thong bao tai khoan</h5>"
                    }

                }
            };
            string Jbody = JsonConvert.SerializeObject(body);
            string url = "https://gateway.gapone.vn/a2p-request";
            using var client = new HttpClient();
            client.DefaultRequestHeaders.Clear();
            client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", token);
            client.DefaultRequestHeaders.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
            var data = new StringContent(Jbody, Encoding.UTF8, "application/json");
            HttpResponseMessage response = await client.PostAsync(url, data);
            response.EnsureSuccessStatusCode();
            var body = await response.Content.ReadAsStringAsync();
            Console.WriteLine(body);
            // send message with fallback, default channel is sms
            var viberMsg = new
            {
                channel = "viber",
                from = new
                {
                    name = "BrandName", 
                },
                to = "84332584400", 
                content = new
                {
                    text = "Nội dung tin nhắn",
                    media = new 
                    {
                        type = "image/jpeg",
                        name = "image",
                        url = "https://icdn.dantri.com.vn/zoom/528_352/2021/10/10/covidhanoi-1633821326759.jpg", 
                    },
                    action = new 
                    {
                        type = "url",
                        text = "Đăng ký ngay",
                        url = "https://my.vnpt.com.vn/ads?a=rRwec324c7",
                    }
                }
            };
            var zaloMsg = new
            {
                channel = "zalo", 
                from = "2293893440987655977",
                to = "84332584400", 

              
                template_id = "21259", 
                template_data = new 
                {
                    customer_name = "Name",
                    OTP = "1234"
                }
            };

            string[] mail = { "nch2699@gmail.com" };
            var emailMsg = new
            {
                channel = "email", 
                from = "name@gmail.vn", 
                to = mail,
                subject = "thong bao tai khoan",
                content = new
                {
                    text = "Thong bao tai khoan",
                    html = "<h5>Thong bao tai khoan</h5>"
                }
            };
            Object[] fallback = { zaloMsg, viberMsg, emailMsg };

            var body = new
            {
                request_id = Guid.NewGuid().ToString(),
                message = new
                {
                    request_id = Guid.NewGuid().ToString(), 
                    channel = "sms", 
                    from = "BrandName", 
                    to = "84332584400", 
                    content = new 
                    {
                        text = "Hello",
                        msg_type = "text", 
                        sms_type = "CSKH" 
                    }
                },
                fallback = fallback

            };
            string Jbody = JsonConvert.SerializeObject(body);
            string url = "https://gateway.gapone.vn/a2p-request";
            using var client = new HttpClient();
            client.DefaultRequestHeaders.Clear();
            client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", token);
            client.DefaultRequestHeaders.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
            var data = new StringContent(Jbody, Encoding.UTF8, "application/json");
            HttpResponseMessage response = await client.PostAsync(url, data);
            response.EnsureSuccessStatusCode();
            var body = await response.Content.ReadAsStringAsync();
            Console.WriteLine(body);
      requestBody:
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/A2PMessage"
            examples:
              example-1:
                value:
                  request_id: 1312rfg3dgdgd
                  message:
                    channel: zalo
                    from: "3123123131"
                    to: "84912345678"
                    template_id: "32311"
                    template_data: {}
                  fallback:
                    - channel: sms
                      from: Gapit
                      to: "84912345678"
                      content:
                        text: " Thong bao tai khoan: account 55875247525, balance 1000000 vnd, date  25/06/2020 11:20. Chi tiet: https://gap1.vn/612345"
                        msg_type: text
                        sms_type: CSKH
        description: Request body
      tags:
        - A2P Message
    parameters: []
  /v1/contacts:
    servers:
      - url: "https://api.gapone.vn"
    get:
      summary: Get All Contacts
      tags:
        - Contact
      responses:
        "200":
          description: OK (Click to see RESPONSE SCHEMA)
          content:
            application/json:
              schema:
                type: object
                properties:
                  data:
                    type: array
                    description: list of contact
                    items:
                      $ref: "#/components/schemas/Contact"
                  has_more:
                    type: boolean
                    description: has more pages
                required:
                  - data
                  - has_more
              examples:
                example-1:
                  value:
                    data:
                      - Email: string
                        Phone: string
                        Skype: string
                        Gender: string
                        Google: string
                        Address: string
                        Twitter: string
                        Facebook: string
                        LastName: string
                        Linkedin: string
                        AvatarUrl: null
                        FirstName: string
                        UserOwner: string
                        DateOfBirth: string
                        TemporaryAddress: string
                    has_more: true
      operationId: get-contact
      description: List all contact
      parameters:
        - schema:
            type: integer
            minimum: 1
          in: query
          name: page
          description: page number
        - schema:
            type: integer
            default: 20
            minimum: 1
            maximum: 100
          in: query
          name: limit
          description: item per page
      x-codeSamples:
        - lang: JavaScript
          source: |
            var axios = require("axios").default;

            var options = {
              method: 'GET',
              url: 'https://gateway.gapone.vn/v1/contacts',
              headers: {'Content-Type': 'application/json', Authorization: 'Bearer {ACCESS_TOKEN}'}
            };

            axios.request(options).then(function (response) {
              console.log(response.data);
            }).catch(function (error) {
              console.error(error);
            });
        - lang: curl
          source: |
            curl --request GET \
              --url https://gateway.gapone.vn/v1/contacts \
              --header 'Authorization: Bearer {ACCESS_TOKEN}' \
              --header 'Content-Type: application/json'
        - lang: C#
          source: |
            var client = new HttpClient();
            var request = new HttpRequestMessage
            {
                Method = HttpMethod.Get,
                RequestUri = new Uri("https://gateway.gapone.vn/v1/contacts"),
                Headers =
                {
                    { "Authorization", "Bearer {ACCESS_TOKEN}" },
                },
            };
            using (var response = await client.SendAsync(request))
            {
                response.EnsureSuccessStatusCode();
                var body = await response.Content.ReadAsStringAsync();
                Console.WriteLine(body);
            }
    post:
      summary: Create a Contact
      operationId: post-contact
      tags:
        - Contact
      responses:
        "201":
          description: Created
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Contact"
              examples:
                example-1:
                  value:
                    LastName: nguyen
                    FirstName: van
                    Phone: "84936123456"
                    Email: nguyenvan@gmail.com
                    Address: Địa chỉ
                    id: 1
      requestBody:
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/Contact"
            examples:
              example-1:
                value:
                  LastName: nguyen
                  FirstName: van
                  Phone: "84936123456"
                  Email: nguyenvan@gmail.com
                  Address: Địa chỉ
      description: |
        Tạo mới khách hàng <br>
        <span style="color:red"> Chú ý: Contact phải chứa ít nhất Email hoặc Phone</span>.

      x-codeSamples:
        - lang: JavaScript
          source: |
            var axios = require("axios").default;

            var options = {
              method: 'POST',
              url: 'https://gateway.gapone.vn/v1/contacts',
              headers: {'Content-Type': 'application/json', Authorization: 'Bearer {ACCESS_TOKEN}'},
              data: {
                LastName: 'nguyen',
                FirstName: 'van',
                Phone: '84936123456',
                Email: 'nguyenvan@gmail.com',
                Address: 'Địa chỉ'
              }
            };

            axios.request(options).then(function (response) {
              console.log(response.data);
            }).catch(function (error) {
              console.error(error);
            });
        - lang: curl
          source: |
            curl --request POST \
              --url https://gateway.gapone.vn/v1/contacts \
              --header 'Authorization: Bearer {ACCESS_TOKEN}' \
              --header 'Content-Type: application/json' \
              --data '{
              "LastName": "nguyen",
              "FirstName": "van",
              "Phone": "84936123456",
              "Email": "nguyenvan@gmail.com",
              "Address": "Địa chỉ"
            }'
        - lang: C#
          source: |
            var client = new HttpClient();
            var request = new HttpRequestMessage
            {
                Method = HttpMethod.Post,
                RequestUri = new Uri("https://gateway.gapone.vn/v1/contacts"),
                Headers =
                {
                    { "Authorization", "Bearer {ACCESS_TOKEN}" },
                },
                Content = new StringContent("{\n  \"LastName\": \"nguyen\",\n  \"FirstName\": \"van\",\n  \"Phone\": \"84936123456\",\n  \"Email\": \"nguyenvan@gmail.com\",\n  \"Address\": \"Địa chỉ\"\n}")
                {
                    Headers =
                    {
                        ContentType = new MediaTypeHeaderValue("application/json")
                    }
                }
            };
            using (var response = await client.SendAsync(request))
            {
                response.EnsureSuccessStatusCode();
                var body = await response.Content.ReadAsStringAsync();
                Console.WriteLine(body);
            }
    parameters: []
  "/v1/contacts/{id}":
    servers:
      - url: "https://api.gapone.vn"
    parameters:
      - schema:
          type: integer
          minimum: 1
        name: id
        in: path
        required: true
    get:
      summary: Get a Contact by Id
      operationId: get-contact-byid
      tags:
        - Contact
      responses:
        "200":
          description: OK
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Contact"
        "":
          description: ""
      x-codeSamples:
        - lang: JavaScript
          source: |
            var options = {
              method: 'GET',
              url: 'https://gateway.gapone.vn/v1/contacts/{id}',
              headers: {'Content-Type': 'application/json', Authorization: 'Bearer {ACCESS_TOKEN}'}
            };

            axios.request(options).then(function (response) {
              console.log(response.data);
            }).catch(function (error) {
              console.error(error);
            });
        - lang: curl
          source: |
            curl --request GET \
              --url https://gateway.gapone.vn/v1/contacts/{id} \
              --header 'Authorization: Bearer {ACCESS_TOKEN}' \
              --header 'Content-Type: application/json'
        - lang: C#
          source: |
            var client = new HttpClient();
            var request = new HttpRequestMessage
            {
                Method = HttpMethod.Get,
                RequestUri = new Uri("https://gateway.gapone.vn/v1/contacts/{id}"),
                Headers =
                {
                    { "Authorization", "Bearer {ACCESS_TOKEN}" },
                },
            };
            using (var response = await client.SendAsync(request))
            {
                response.EnsureSuccessStatusCode();
                var body = await response.Content.ReadAsStringAsync();
                Console.WriteLine(body);
            }
    patch:
      summary: Update a Contact
      operationId: patch-contacts
      tags:
        - Contact
      responses:
        "200":
          description: OK
      requestBody:
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/Contact"
            examples:
              example-1:
                value:
                  Email: string
                  Phone: string
                  LastName: string
                  FirstName: string
      x-codeSamples:
        - lang: JavaScript
          source: |
            var axios = require("axios").default;

            var options = {
              method: 'PATCH',
              url: 'https://gateway.gapone.vn/v1/contacts/{id}',
              headers: {'Content-Type': 'application/json', Authorization: 'Bearer {ACCESS_TOKEN}'},
              data: {Email: 'string', Phone: 'string', LastName: 'string', FirstName: 'string'}
            };

            axios.request(options).then(function (response) {
              console.log(response.data);
            }).catch(function (error) {
              console.error(error);
            });
        - lang: curl
          source: |
            curl --request PATCH \
              --url https://gateway.gapone.vn/v1/contacts/{id} \
              --header 'Authorization: Bearer {ACCESS_TOKEN}' \
              --header 'Content-Type: application/json' \
              --data '{
              "Email": "string",
              "Phone": "string",
              "LastName": "string",
              "FirstName": "string"
            }'
        - lang: C#
          source: |
            var client = new HttpClient();
            var request = new HttpRequestMessage
            {
                Method = HttpMethod.Patch,
                RequestUri = new Uri("https://gateway.gapone.vn/v1/contacts/{id}"),
                Headers =
                {
                    { "Authorization", "Bearer {ACCESS_TOKEN}" },
                },
                Content = new StringContent("{\n  \"Email\": \"string\",\n  \"Phone\": \"string\",\n  \"LastName\": \"string\",\n  \"FirstName\": \"string\"\n}")
                {
                    Headers =
                    {
                        ContentType = new MediaTypeHeaderValue("application/json")
                    }
                }
            };
            using (var response = await client.SendAsync(request))
            {
                response.EnsureSuccessStatusCode();
                var body = await response.Content.ReadAsStringAsync();
                Console.WriteLine(body);
            }
    delete:
      summary: Delete a Contact
      operationId: delete-contacts-id
      tags:
        - Contact
      responses:
        "204":
          description: No Content
      x-codeSamples:
        - lang: JavaScript
          source: |
            var axios = require("axios").default;

            var options = {
              method: 'DELETE',
              url: 'https://gateway.gapone.vn/v1/contacts/{id}',
              headers: {'Content-Type': 'application/json', Authorization: 'Bearer {ACCESS_TOKEN}'}
            };

            axios.request(options).then(function (response) {
              console.log(response.data);
            }).catch(function (error) {
              console.error(error);
            });
        - lang: curl
          source: |
            curl --request DELETE \
              --url https://gateway.gapone.vn/v1/contacts/{id} \
              --header 'Authorization: Bearer {ACCESS_TOKEN}' \
              --header 'Content-Type: application/json'
        - lang: C#
          source: |
            var client = new HttpClient();
            var request = new HttpRequestMessage
            {
                Method = HttpMethod.Delete,
                RequestUri = new Uri("https://gateway.gapone.vn/v1/contacts/{id}"),
                Headers =
                {
                    { "Authorization", "Bearer {ACCESS_TOKEN}" },
                },
            };
            using (var response = await client.SendAsync(request))
            {
                response.EnsureSuccessStatusCode();
                var body = await response.Content.ReadAsStringAsync();
                Console.WriteLine(body);
            }
  /segments:
    get:
      summary: List all Segments
      tags:
        - Segment
      responses:
        "200":
          description: OK (Click to see RESPONSE SCHEMA)
          headers: {}
          content:
            application/json:
              schema:
                type: object
                properties:
                  data:
                    type: array
                    items:
                      $ref: "#/components/schemas/Segment"
                  has_more:
                    type: boolean
                required:
                  - data
                  - has_more
              examples:
                example-1:
                  value:
                    data:
                      - id: 1
                        name: VIP
                        description: VIP user
                        filters:
                          - - operator: Equal
                              property: email
                              value: test@gapone.vn
                    has_more: true
      operationId: get-segments
      description: ""
      security:
        - OAuth2ClientCredsAuth: []
        - BasicAuth: []
    post:
      summary: Creat a Segment
      operationId: post-segments
      responses:
        "201":
          description: Created
      description: ""
      security:
        - OAuth2ClientCredsAuth: []
        - BasicAuth: []
      requestBody:
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/Segment"
            examples:
              example-1:
                value:
                  name: VIP
                  description: VIP user
                  filters:
                    - - operator: Equal
                        property: email
                        value: test@gapone.vn
      tags:
        - Segment
  "/segments/{id}":
    parameters:
      - schema:
          type: integer
          minimum: 1
        name: id
        in: path
        required: true
        description: sengment Id
    get:
      summary: Get a Segment by Id
      tags:
        - Segment
      responses: {}
      operationId: get-segments-id
      security:
        - OAuth2ClientCredsAuth: []
        - BasicAuth: []
      requestBody:
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/Segment"
            examples:
              example-1:
                value:
                  id: 1
                  name: VIP
                  description: VIP user
                  filters:
                    - - operator: Equal
                        property: email
                        value: test@gapone.vn
    delete:
      summary: Delete a segment by id
      operationId: delete-segments-id
      responses:
        "204":
          description: No Content
      security:
        - OAuth2ClientCredsAuth: []
        - BasicAuth: []
      tags:
        - Segment
    patch:
      summary: Update a Segment
      operationId: patch-segments-id
      responses:
        "200":
          description: OK
      security:
        - OAuth2ClientCredsAuth: []
        - BasicAuth: []
      requestBody:
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/Segment"
            examples:
              example-1:
                value:
                  name: VIP
                  description: VIP user
                  filters:
                    - - operator: Equal
                        property: email
                        value: test@gapone.vn
      tags:
        - Segment
  /status-query:
    post:
      summary: Lấy trạng thái gửi tin
      operationId: post-status-query
      responses:
        "200":
          description: OK (Click to see RESPONSE SCHEMA)
          content:
            application/json:
              schema:
                type: array
                uniqueItems: true
                minItems: 1
                items:
                  $ref: "#/components/schemas/StatusQueryResponse"
              examples:
                example-1:
                  value:
                    - message_id: 81f13950-19ca-11eb-94ac-35235df80632
                      status:
                        - channel: viber
                          sent_status: -1
                          sent_time: "2020-10-29T09:38:31.925Z"
                        - channel: sms
                          sent_status: 0
                          sent_time: "2020-10-29T09:38:32.584Z"
                          delivery_status: 0
                          delivery_time: "2020-10-29T09:38:32.683Z"
                    - message_id: 5545ea50-249c-5e4b-dadc-693dfa598051
        "400":
          description: Bad Request. Kiểm tra tham số request. Không retry.
        "401":
          description: Unauthorized. Kiểm tra tham số authentication (client_id & client_secret). Không retry.
        "403":
          description: Forbidden. Kiểm tra tham số đường dẫn và quyền truy cập. Không retry.
        "422":
          description: Unprocessable. Nội dung request không hợp lệ. Kiểm tra lại tham số tin nhắn. Không retry.
        "429":
          description: Too many requests. Tốc độ gửi request quá cao. Cần giảm tốc độ gửi requests xuống mức cho phép (default 100 TPS).
        "503":
          description: Server busy. Thử lại sau 5-10 giây.
        4xx:
          description: Bad Request. Kiểm tra tham số request. Không retry.
        5xx:
          description: Lỗi server. Thử lại sau 5-10 giây. Liên hệ Customer Support.
      tags:
        - A2P Message
      description: |-
        Lấy trạng thái gửi tin theo message_id.
        Cơ chế xác thực sử dụng HTTP Bearer auhentication scheme, với token là giá trị access_token
        nhận được từ Auth-API server ở [bước trên](#section/Authentication/OAuth2ClientCredsAuth) .
      security:
        - OAuth2ClientCredsAuth: []
        - BasicAuth: []
      requestBody:
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/StatusQuery"
            examples:
              example-1:
                value:
                  message_id:
                    - 81f13950-19ca-11eb-94ac-35235df80632
                    - 5545ea50-249c-5e4b-dadc-693dfa598051
      x-codeSamples:
        - lang: JavaScript
          source: |
            var axios = require("axios").default;

            var options = {
              method: 'POST',
              url: 'https://gateway.gapone.vn/status-query',
              headers: {'Content-Type': 'application/json', Authorization: 'Bearer {ACCESS_TOKEN}'},
              data: {
                message_id: ['81f13950-19ca-11eb-94ac-35235df80632', '5545ea50-249c-5e4b-dadc-693dfa598051']
              }
            };

            axios.request(options).then(function (response) {
              console.log(response.data);
            }).catch(function (error) {
              console.error(error);
            });
        - lang: curl
          source: |
            curl --request POST \
            --url https://gateway.gapone.vn/status-query \
            --header 'Authorization: Bearer {ACCESS_TOKEN}' \
            --header 'Content-Type: application/json' \
            --data '{
            "message_id": [
              "81f13950-19ca-11eb-94ac-35235df80632",
              "5545ea50-249c-5e4b-dadc-693dfa598051"
            ]
            }'
        - lang: C#
          source: |
            var client = new HttpClient();
            var request = new HttpRequestMessage
            {
                Method = HttpMethod.Post,
                RequestUri = new Uri("https://gateway.gapone.vn/status-query"),
                Headers =
                {
                    { "Authorization", "Bearer {ACCESS_TOKEN}" },
                },
                Content = new StringContent("{\n  \"message_id\": [\n    \"81f13950-19ca-11eb-94ac-35235df80632\",\n    \"5545ea50-249c-5e4b-dadc-693dfa598051\"\n  ]\n}")
                {
                    Headers =
                    {
                        ContentType = new MediaTypeHeaderValue("application/json")
                    }
                }
            };
            using (var response = await client.SendAsync(request))
            {
                response.EnsureSuccessStatusCode();
                var body = await response.Content.ReadAsStringAsync();
                Console.WriteLine(body);
            }
      x-extension-2: null
  "/v1/contacts/email/{email}":
    servers:
      - url: "https://api.gapone.vn"
    get:
      summary: Get Contact by Email
      tags:
        - Contact
      responses:
        "200":
          description: OK
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Contact"
              examples:
                example-1:
                  value:
                    id: 1
                    Email: user@example.com
                    Phone: "84936123456"
                    Skype: string
                    Gender: string
                    Google: string
                    Address: string
                    Twitter: string
                    Facebook: string
                    LastName: string
                    Linkedin: string
                    AvatarUrl: string
                    FirstName: string
                    UserOwner: string
                    DateOfBirth: string
                    TemporaryAddress: string
      operationId: "get-v1-contacts-email-:email"
      description: ""
    parameters:
      - schema:
          type: string
          format: email
        name: email
        in: path
        required: true
  "/v1/contacts/phone/{phone}":
    servers:
      - url: "https://api.gapone.vn"
    get:
      summary: Get Contact by phone
      tags:
        - Contact
      responses:
        "200":
          description: OK
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Contact"
              examples:
                example-1:
                  value:
                    Email: khachuy@gapit.com.vn
                    Phone: "84969533045"
                    Skype: khachuy192
                    Gender: Male
                    Google: ""
                    Address: Hà Nội
                    Twitter: ""
                    Facebook: "https://www.facebook.com/buikhachuy192"
                    LastName: Lê Mạnh
                    Linkedin: ""
                    AvatarUrl: null
                    FirstName: Hùng
                    UserOwner: bd7b7759-0bcb-4edf-aeaf-7822dcf8d7e0
                    DateOfBirth: 15/06/1976
                    TemporaryAddress: Hà Nội
      operationId: get-v1-contacts-phone
      description: ""
    parameters:
      - schema:
          type: string
          example: "84969533045"
        name: phone
        in: path
        required: true
  /v1/contacts/batch:
    servers:
      - url: "https://api.gapone.vn"
    post:
      summary: Create a batch of contacts
      operationId: post-v1-contacts-batch
      responses:
        "200":
          description: OK
          content:
            application/json:
              schema:
                type: array
                items:
                  type: object
                  properties:
                    id:
                      type: integer
                      description: contact id
                      minimum: 1
                      example: 1
                  required:
                    - id
              examples:
                example-1:
                  value:
                    - id: 1
      requestBody:
        content:
          application/json:
            schema:
              type: array
              minItems: 1
              maxItems: 100
              description: List of contacts
              items:
                $ref: "#/components/schemas/Contact"
            examples:
              example-1:
                value:
                  - Email: khachuy@gapit.com.vn
                    Phone: "84969533045"
                    Skype: khachuy192
                    Gender: Male
                    Google: ""
                    Address: Hà Nội
                    Twitter: ""
                    Facebook: "https://www.facebook.com/buikhachuy192"
                    LastName: Lê Mạnh
                    Linkedin: ""
                    AvatarUrl: null
                    FirstName: Hùng
                    UserOwner: bd7b7759-0bcb-4edf-aeaf-7822dcf8d7e0
                    DateOfBirth: 15/06/1976
                    TemporaryAddress: Hà Nội
      tags:
        - Contact
  /v1/contacts/properties/list:
    servers:
      - url: "https://api.gapone.vn"
    get:
      summary: List Contact properties
      tags:
        - Contact
      responses:
        "200":
          description: OK
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: "#/components/schemas/Property"
      operationId: get-v1-contacts-properties-list
  /v1/products:
    servers:
      - url: "https://api.gapone.vn"
    post:
      summary: ""
      operationId: Create Product
      responses:
        "201":
          description: Created
          content:
            application/json:
              schema:
                description: ""
                type: object
                x-examples:
                  example-1:
                    id: 1170
                    variants:
                      - id: 2208
                properties:
                  id:
                    type: number
                    description: Gapone product id
                  variants:
                    type: array
                    uniqueItems: true
                    minItems: 1
                    items:
                      type: object
                      properties:
                        id:
                          type: number
                          description: Gapone variant id
                      required:
                        - id
                required:
                  - id
                  - variants
      security:
        - OAuth2ClientCredsAuth: []
        - BasicAuth: []
      parameters: []
      requestBody:
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/Product"
            examples:
              example-1:
                value:
                  property:
                    code: AONAM-1111
                    name: Áo Nao
                  variants:
                    - sku: AONAM-1111-S
                      price: 2542000
                      title: Cỡ S
                      quantity: 1
                    - sku: AONAM-1111-M
                      price: 2542000
                      title: Cỡ M
                      quantity: 1
      description: Tạo mới sản phẩm
      tags:
        - Product
  "/v1/products/code/{code}":
    servers:
      - url: "https://api.gapone.vn"
    get:
      summary: Get product by code
      tags:
        - Product
      responses:
        "200":
          description: OK
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Product"
              examples:
                example-1:
                  value:
                    id: 1
                    property:
                      code: string
                      name: string
                    variants:
                      sku: string
                      price: 0
                      title: string
                      quantity: 0
      operationId: Get product by code
      description: lấy thông tin sản phẩm theo mã(code)
      security:
        - OAuth2ClientCredsAuth: []
        - BasicAuth: []
    parameters:
      - schema:
          type: string
        name: code
        in: path
        required: true
        description: mã sản phẩm
  /v1/deals:
    servers:
      - url: "https://api.gapone.vn"
    post:
      summary: ""
      operationId: Create Deal
      responses:
        "201":
          description: Created
          content:
            application/json:
              schema:
                description: ""
                type: object
                x-examples:
                  example-1:
                    id: 1
                properties:
                  id:
                    type: number
                    description: GAPONE id
                required:
                  - id
              examples:
                example-1:
                  value:
                    id: 1
      description: Tạo mới đơn hàng
      security:
        - OAuth2ClientCredsAuth: []
        - BasicAuth: []
      requestBody:
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/Deal"
            examples:
              example-1:
                value:
                  contact_id: 1
                  property:
                    total: 22290000
                    discount: 0
                    quantity: 1
                    ordered_at: 1635766200
                    total_order: 22290000
                    order_number: Order12345
                    order_source: Lazada
                    order_status: completed_payment
                    transport_fee: 0
                  line_items:
                    - sku: "15325675246"
                      price: 22290000
                      title: 160x300x34
                      total: 22290000
                      quantity: 1
                      product_id: 361
                      variant_id: 679
                      product_name: Nệm cao su
      tags:
        - Deal
  "/v1/deals/{id}":
    servers:
      - url: "https://api.gapone.vn"
    patch:
      summary: ""
      operationId: Update deal status
      responses:
        "200":
          description: OK
      description: Cập nhật trạng thái đơn hàng
      security:
        - OAuth2ClientCredsAuth: []
        - BasicAuth: []
      requestBody:
        content:
          application/json:
            schema:
              description: ""
              type: object
              x-examples:
                example-1:
                  property:
                    updated_at: 1635766200
                    order_status: completed_payment
              properties:
                property:
                  type: object
                  required:
                    - updated_at
                    - order_status
                  properties:
                    updated_at:
                      type: number
                      description: "thời điểm cập nhật trạng thái đơn hàng, unix timestamp"
                    order_status:
                      type: string
                      minLength: 1
                      description: "Trạng thái đơn hàng,[**danh sách tại đây**](#tag/order_status)"
              required:
                - property
            examples:
              example-1:
                value:
                  property:
                    updated_at: 1635766200
                    order_status: completed_payment
        description: ""
      tags:
        - Deal
    parameters:
      - schema:
          type: number
        name: id
        in: path
        required: true
        description: id GAPONE của đơn hàng
  "/v1/deals/trans/{order_number}":
    servers:
      - url: "https://api.gapone.vn"
    get:
      summary: Get Deal by Order number
      tags:
        - Deal
      responses:
        "200":
          description: OK
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Deal"
              examples:
                example-1:
                  value:
                    id: 0
                    contact_id: 0
                    property:
                      total: 0
                      discount: 0
                      quantity: 0
                      ordered_at: 0
                      total_order: 0
                      order_number: string
                      order_source: string
                      order_status: string
                      transport_fee: 0
                    line_items:
                      - sku: string
                        price: 0
                        title: string
                        total: 0
                        quantity: 0
                        product_id: 0
                        variant_id: 0
                        product_name: string
      operationId: Get Deal by Order number
      description: Lấy thông tin đơn hàng theo mã đơn
      security:
        - OAuth2ClientCredsAuth: []
        - BasicAuth: []
    parameters:
      - schema:
          type: string
        name: order_number
        in: path
        required: true
        description: mã đơn hàng
  "/v1/zns/oas":
    servers:
      - url: "https://api.gapone.vn"
    get:
      summary: Get All Zalo official account (Zalo OA)
      tags:
        - Zalo
      responses:
        "200":
          description: OK
          content:
            application/json:
              schema:
                type: object
              examples:
                example-1:
                  value: [{ "oa_id": "123131231432423", "oa_name": "GAPIT" }]

      security:
        - OAuth2ClientCredsAuth: []
        - BasicAuth: []
  "/v1/zns/oas/{oa_id}/templates":
    servers:
      - url: "https://api.gapone.vn"
    get:
      summary: Get All ZNS templates of OA
      tags:
        - Zalo
      responses:
        "200":
          description: OK
          content:
            application/json:
              schema:
                type: object
              examples:
                example-1:
                  value:
                    [
                      {
                        "oa_id": "12345678",
                        "template_id": "1234",
                        "name": "Thông báo cảm ơn khách hàng đã sử dụng dịch vụ",
                        "params": "ma_KH,ten_KH",
                        "type_template": "TEXT",
                        "template_object":
                          {
                            "status": "ENABLE",
                            "timeout": 7200000,
                            "listParams":
                              [
                                {
                                  "name": "ten_KH",
                                  "type": "STRING",
                                  "require": true,
                                  "maxLength": 30,
                                  "minLength": 0,
                                  "acceptNull": false,
                                },
                                {
                                  "name": "ma_KH",
                                  "type": "STRING",
                                  "require": true,
                                  "maxLength": 30,
                                  "minLength": 0,
                                  "acceptNull": false,
                                },
                              ],
                            "previewUrl": "https://account.zalo.cloud/znspreview/PshcN5E49TRDsHGnSirOVQ==",
                            "templateId": 1234,
                            "templateTag": "IN_TRANSACTION",
                            "templateName": "Thông báo cảm ơn khách hàng đã sử dụng dịch vụ",
                            "templateQuality": "UNDEFINED",
                            "applyTemplateQuota": false,
                          },
                      },
                    ]

      security:
        - OAuth2ClientCredsAuth: []
        - BasicAuth: []
      parameters:
        - schema:
            type: string
          name: oa_id
          in: path
          required: true
          description: official account id
  "/v1/zns/rating":
    servers:
      - url: "https://api.gapone.vn"
    get:
      summary: Get customer feedback for ZNS rating templates
      tags:
        - Zalo
      responses:
        "200":
          description: OK
          content:
            application/json:
              schema:
                type: object

      security:
        - OAuth2ClientCredsAuth: []
        - BasicAuth: []
      parameters:
        - schema:
            type: string
          name: template_id
          in: query
          required: true
          description: zns template id
        - schema:
            type: integer
          in: query
          name: from_time
          required: true
          description: The starting time to get rating, unix timestamp
        - schema:
            type: integer
            minimum: 1
          in: query
          name: page
          description: page number
        - schema:
            type: integer
            default: 20
            minimum: 1
            maximum: 100
          in: query
          name: limit
          description: item per page
  "/v1/zns/{oa_id}/journey/{phone}?token_type={token_type}":
    servers:
      - url: "https://api.gapone.vn"
    post:
      summary: Create ZNS journey token
      description: Lấy token để gửi tin ZNS hành trình, xem thêm [tại đây](https://developers.zalo.me/docs/zalo-notification-service/gui-tin-zns/gui-zns-journey)
      tags:
        - Zalo
      responses:
        "200":
          description: OK
          content:
            application/json:
              schema:
                type: object
              examples:
                example-1:
                  value:
                    {
                      "error": 0,
                      "message": "Success",
                      "token": "token",
                      "journey_id": "id",
                    }

      security:
        - OAuth2ClientCredsAuth: []
        - BasicAuth: []
      parameters:
        - schema:
            type: string
          name: oa_id
          in: path
          required: true
          description: official account id
        - schema:
            type: string
          name: phone
          in: path
          required: true
          description: user phone number
        - schema:
            type: string
            enum:
              - token_logistics_7
              - token_logistics_30
              - token_coach_bus_7
              - token_coach_bus_30
          name: token_type
          in: query
          required: true
          description: journey type
  "/v1/zns/rsa":
    servers:
      - url: "https://api.gapone.vn"
    post:
      summary: Create ZNS RSA public key
      description: Tạo public key để gửi tin ZNS mã hóa, xem thêm [tại đây](https://developers.zalo.me/docs/zalo-notification-service/gui-tin-zns/gui-zns-voi-he-ma-hoa-rsa)
      tags:
        - Zalo
      responses:
        "200":
          description: OK
          content:
            application/json:
              schema:
                type: object
              examples:
                example-1:
                  value:
                    {
                      "data": { "public_key": "e6fa968f027" },
                      "error": 0,
                      "message": "Success",
                    }

      security:
        - OAuth2ClientCredsAuth: []
        - BasicAuth: []
      parameters:
        - schema:
            type: string
          name: oa_id
          in: path
          required: true
          description: official account id
    get:
      summary: Get ZNS RSA public key
      description: Lấy public key để gửi tin ZNS mã hóa, xem thêm [tại đây](https://developers.zalo.me/docs/zalo-notification-service/gui-tin-zns/gui-zns-voi-he-ma-hoa-rsa)
      tags:
        - Zalo
      responses:
        "200":
          description: OK
          content:
            application/json:
              schema:
                type: object
              examples:
                example-1:
                  value:
                    {
                      "data": { "public_key": "e6fa968f027" },
                      "error": 0,
                      "message": "Success",
                    }

      security:
        - OAuth2ClientCredsAuth: []
        - BasicAuth: []
      parameters:
        - schema:
            type: string
          name: oa_id
          in: path
          required: true
          description: official account id
  "/v1/sender/sms":
    servers:
      - url: "https://api.gapone.vn"
    get:
      summary: List SMS Brandname (SMS Sender)
      tags:
        - Brandname
      responses:
        "200":
          description: OK
          content:
            application/json:
              schema:
                type: object
              examples:
                example-1:
                  value: [{ "id": 10, "name": "GAPIT" }]
      security:
        - OAuth2ClientCredsAuth: []
        - BasicAuth: []
  "/v1/sender/viber":
    servers:
      - url: "https://api.gapone.vn"
    get:
      summary: List Viber Brandname (Viber Sender)
      tags:
        - Brandname
      responses:
        "200":
          description: OK
          content:
            application/json:
              schema:
                type: object
              examples:
                example-1:
                  value: [{ "id": 10, "name": "GAPIT" }]
      security:
        - OAuth2ClientCredsAuth: []
        - BasicAuth: []
  "/v1/sender/whatsapp":
    servers:
      - url: "https://api.gapone.vn"
    get:
      summary: List Whatsapp sender number
      tags:
        - Brandname
      responses:
        "200":
          description: OK
          content:
            application/json:
              schema:
                type: object
              examples:
                example-1:
                  value:
                    [
                      {
                        "id": 1,
                        "name": "Gapit ",
                        "phone_number": "12155238886",
                      },
                    ]
  "/v1/sender/whatsapp/{phone_number}/templates":
    servers:
      - url: "https://api.gapone.vn"
    get:
      summary: List Whatsapp sender templates
      parameters:
        - name: phone_number
          in: path
          required: true
          description: Phone number of the Whatsapp sender
          schema:
            type: string
      tags:
        - Brandname
      responses:
        "200":
          description: OK
          content:
            application/json:
              schema:
                type: object
              examples:
                example-1:
                  value:
                    [
                      {
                          "template_id": "HX123sdsf2342342",
                          "template_name": "new_2",
                          "params": [
                              "1"
                          ],
                          "message_content": {
                          
                          }
                      }
                  ]  
      security:
        - OAuth2ClientCredsAuth: []
        - BasicAuth: []
components:
  schemas:
    SentResponse:
      title: SentResponse
      type: object
      x-examples:
        example-1:
          message_id: 4172d060-937d-11ea-8dd3-83fb5ae4793c
          request_id: 1312rfg3dgdgd
      description: Sent Message response
      properties:
        message_id:
          type: string
          description: gapone message id
          example: 4172d060-937d-11ea-8dd3-83fb5ae4793c
        request_id:
          type: string
          example: 1312rfg3dgdgd
      required:
        - message_id
        - request_id
    SMS:
      title: SMS
      type: object
      x-examples:
        example-2:
          channel: sms
          from: Brandname
          to: "84912345678"
          content:
            text: " Thong bao tai khoan: account 55875247525, balance 1000000 vnd, date  25/06/2020 11:20. Chi tiet: https://gap1.vn/612345"
            msg_type: text
            sms_type: CSKH
      properties:
        channel:
          type: string
          enum:
            - sms
          description: sms channel
        from:
          type: string
          example: Brandname
          description: Brandname (Liên hệ Gapit để đăng ký), lấy danh sách Brandname [tại đây](#tag/Brandname/paths/~1v1~1sender~1sms/get)
        to:
          type: string
          example: "84912345678"
          description: recipient number
        content:
          type: object
          required:
            - text
            - msg_type
            - sms_type
          description: ""
          properties:
            text:
              type: string
              description: |
                sms content<br>
                Giới hạn số lượng ký tự theo từng msg_type:<br>
                text: VIETTEL - 799, VINAPHONE/I-TELECOM/MOBIFONE/VIETNAMOBILE/GMOBILE/REDDI - 612<br>
                unicode: VIETTEL - 603, VINAPHONE/I-TELECOM/REDDI - 268, MOBIFONE/VIETNAMOBILE/GMOBILE - 335<br>
              example: " Thong bao tai khoan: account 55875247525, balance 1000000 vnd, date  25/06/2020 11:20. Chi tiet: https://gap1.vn/612345"
            msg_type:
              type: string
              enum:
                - text
                - unicode
              example: text
              description: sms content type
            sms_type:
              type: string
              enum:
                - CSKH
                - QC
              description: sms type
      required:
        - channel
        - from
        - to
        - content
    Zalo:
      title: ZNS
       
      type: object
      x-examples:
        example-1:
          channel: zalo
          from: "3123123131"
          to: "84912345678"
          template_id: "32311"
          template_data:
            title: Thông báo tài khoản
            account: "55875247525"
            balance: "1000000"
            date: "25/06/2020 11:20"
      properties:
        channel:
          type: string
          enum:
            - zalo
          description: zalo channel
        from:
          type: string
          example: "3123123131"
          description: zalo official account ID, get Id from [here](#tag/Zalo/paths/~1v1~1zns~1oas/get)
        to:
          type: string
          example: "84912345678"
          description: |
            Số điện thoại người nhận.
            Hỗ trợ 2 định dạng:
            1. Số điện thoại quốc tế : "84912345678" hoặc "+84912345678"
            2. SHA-256(số điện thoại quốc tế) để tăng tính bảo mật: "114d70ba7d62d461c356beb9e710b7a95580f2df57dd61882c4008a9cf7be34a"
        template_id:
          type: string
          example: "32311"
          description: zalo template id, get from [here](#tag/Zalo/paths/~1v1~1zns~1oas/get)
        template_data:
          description: template custom data, get from [here](#tag/Zalo/paths/~1v1~1zns~1oas~1{oa_id}/get)
          type: object
      required:
        - channel
        - from
        - to
        - template_id
        - template_data
    Zalo_Journey:
      title: ZNS hành trình
      type: object
      x-examples:
        example-1:
          channel: zalo
          from: "3123123131"
          to: "84912345678"
          template_id: "32311"
          template_data:
            title: Thông báo tài khoản
            account: "55875247525"
            balance: "1000000"
            date: "25/06/2020 11:20"
          journey_token: "123124rgdr314efe32423"
      properties:
        channel:
          type: string
          enum:
            - zalo
          description: zalo channel
        from:
          type: string
          example: "3123123131"
          description: zalo official account ID, get Id from [here](#tag/Zalo/paths/~1v1~1zns~1oas/get)
        to:
          type: string
          example: "84912345678"
          description: |
            Số điện thoại người nhận.
            Hỗ trợ 2 định dạng:
            1. Số điện thoại quốc tế : "84912345678" hoặc "+84912345678"
            2. SHA-256(số điện thoại quốc tế) để tăng tính bảo mật: "114d70ba7d62d461c356beb9e710b7a95580f2df57dd61882c4008a9cf7be34a"
        template_id:
          type: string
          example: "32311"
          description: zalo template id, get from [here](#tag/Zalo/paths/~1v1~1zns~1oas/get)
        template_data:
          description: template custom data, get from [here](#tag/Zalo/paths/~1v1~1zns~1oas~1{oa_id}/get)
          type: object
        journey_token:
          description: Chỉ yêu cầu khi gửi tin [ZNS hành trình, tìm hiểu tại đây](#tag/Zalo_Journey_Channel), [Tạo journey token tại đây](#tag/Zalo/paths/~1v1~1zns~1{oa_id}~1journey~1{phone}?token_type={token_type}/post)
          type: string
      required:
        - channel
        - from
        - to
        - template_id
        - template_data

    Zalo_RSA:
      title: ZNS Mã hóa
      type: object
      externalDocs:
        description: Find more info here
        url: https://example.com
      x-examples:
        example-1:
          channel: zalo
          from: "3123123131"
          to: "849123456"
          template_id: "200259"
          rsa: true
          template_data:
            param1: wWvo6ofNrjH7HeMrDb9WXnWbA5XJaqKnm0hPG7oEquLrUE8FDEP6oFr7h/YPlGjyo97GlkYZufL9doJU5p2hlf fn253rA0SRZWvVxBMr/haW7oca5FQs1O5jyrLeSDwT3qujieaR95VzbReUWgy2RqbC/FC3z/1Gi5ThTdIhVFw=
      properties:
        channel:
          type: string
          enum:
            - zalo
          description: zalo channel
        from:
          type: string
          example: "3123123131"
          description: zalo official account ID
        to:
          type: string
          example: "849123456"
          description: |
            Số điện thoại người nhận.
            Hỗ trợ 2 định dạng:
            1. Số điện thoại quốc tế : "84912345678" hoặc "+84912345678"
            2. Số điện thoại quốc tế  được mã hóa
        rsa:
          type: bool
          example: true
          description: "gửi zns mã hóa, [tìm hiểu tại đây](#tag/Zalo_RSA_Channel)"
        template_id:
          type: string
          example: "200259"
          description: zalo template id (Liên hệ Gapit để đăng ký)
        template_data:
          description: template custom data được mã hóa các tham số
          type: object
      required:
        - channel
        - from
        - to
        - template_id
        - template_data
    ZaloOA:
      title: Zalo OA
      type: object
      x-examples:
        example-1:
          channel: zalooa
          from: "227507149430042342"
          recipient:
            user_id: "12321312312"
          template_id: "32311"
          content:
            text: Hello
      properties:
        channel:
          type: string
          enum:
            - zalooa
          description: zalo oa channel
        from:
          type: string
          example: "227507149430042342"
          description: zalo official account ID
        recipient:
          type: object
          description: |
            zalo oa user id
          properties:
            user_id:
              type: string
              example: 12321312312
              description: zalo user id
          required:
            - user_id
        content:
          description: "Nội dung gửi tin, cấu trúc giống trường message tài liệu [Official Account API](https://developers.zalo.me/docs/api/official-account-api/gui-tin-va-thong-bao-qua-oa/v3/tong-quan/tong-quan-post-7047) của Zalo, hỗ trợ các loại tin [văn bản](https://developers.zalo.me/docs/api/official-account-api/gui-tin-va-thong-bao-qua-oa/gui-thong-bao-van-ban-post-5072), [hình ảnh](https://developers.zalo.me/docs/api/official-account-api/gui-tin-va-thong-bao-qua-oa/gui-thong-bao-theo-mau-dinh-kem-anh-post-5068), [danh sách](https://developers.zalo.me/docs/api/official-account-api/gui-tin-va-thong-bao-qua-oa/gui-thong-bao-theo-mau-dinh-kem-danh-sach-post-5064)"
          type: object
          example: { "text": "hello" }
        message_type:
          description: "Loại tin OA theo [V3](https://developers.zalo.me/docs/api/official-account-api/gui-tin-va-thong-bao-qua-oa/v3/tong-quan/tong-quan-post-7047) , chấp nhận 1 trong 3 giá trị sau: consulting (tin tư vấn), transaction (tin giao dịch), promotion (tin truyền thông)"
          type: string
          enum:
            - consulting
            - transaction
            - promotion
      required:
        - channel
        - from
        - recipient
        - content
    Viber:
      title: Viber
      type: object
      properties:
        channel:
          type: string
          description: |
            viber channel
          enum:
            - viber
        from:
          description: ""
          type: object
          properties:
            name:
              type: string
              example: Brandname
              description: Brandname (Liên hệ Gapit để đăng ký), lấy danh sách Brandname [tại đây](#tag/Brandname/paths/~1v1~1sender~1viber/get)
          required:
            - name
        to:
          type: string
          example: "84912345678"
          description: viber recipient number
        content:
          oneOf:
            - $ref: "#/components/schemas/ViberLegacyContent"
            - $ref: "#/components/schemas/ViberContent"
      required:
        - channel
        - to
        - content
        - action
      x-examples:
        example-1:
          channel: viber
          from:
            name: Brandname
          to: "84912345678"
          content:
            text: "Thong bao tai khoan: account 55875247525, balance 1000000 vnd, date  25/06/2020 11:20"
            media:
              type: image/jpg
              name: image
              url: "https://bit.ly/1234256"
          action:
            type: url
            text: Visit
            url: "https://gapone.vn"
    ViberLegacyContent:
      description: (click the property name for details)
      type: object
      required:
        - text
        - media
      properties:
        text:
          type: string
          example: "Thong bao tai khoan: account 55875247525, balance 1000000 vnd, date  25/06/2020 11:20"
          description: viber text content <br> Giới hạn 1000 ký tự
        media:
          type: object
          required:
            - type
            - url
          description: viber media content (click the property name for details)
          properties:
            type:
              type: string
              example: image/jpg
              description: media content type
            name:
              type: string
              example: image
              description: media name
            url:
              type: string
              format: uri
              example: "https://bit.ly/1234256"
              description: media resource url <br> Giới hạn 255 ký tự
        action:
          type: object
          description: viber button action (click the property name for details)
          required:
            - type
            - text
            - url
          properties:
            type:
              type: string
              enum:
                - url
              description: |
                action type
            text:
              type: string
              example: Visit
              description: button caption <br> Giới hạn 20 ký tự
            url:
              type: string
              example: "https://gapone.vn"
              description: action url <br> Giới hạn 255 ký tự
    ViberBaseContent:
      type: object
      required:
        - message_type
      properties:
        message_type:
          type: string
          enum: [TEXT, TEXT_TEMPLATE, IMG, FILE, VIDEO, PREDEFINED_TEMPLATE]
    ViberContent:
      oneOf:
        - $ref: "#/components/schemas/ViberTextMessage"
        - $ref: "#/components/schemas/ViberTextTemplateMessage"
        - $ref: "#/components/schemas/ViberImageMessage"
        - $ref: "#/components/schemas/ViberFileMessage"
        - $ref: "#/components/schemas/ViberVideoMessage"
        - $ref: "#/components/schemas/ViberPredefinedTemplateMessage"
      discriminator:
        propertyName: message_type
        mapping:
          TEXT: "#/components/schemas/ViberTextMessage"
          TEXT_TEMPLATE: "#/components/schemas/ViberTextTemplateMessage"
          IMG: "#/components/schemas/ViberImageMessage"
          FILE: "#/components/schemas/ViberFileMessage"
          VIDEO: "#/components/schemas/ViberVideoMessage"
          PREDEFINED_TEMPLATE: "#/components/schemas/ViberPredefinedTemplateMessage"
    ViberTextMessage:
      allOf:
        - $ref: "#/components/schemas/ViberBaseContent"
        - type: object
          required:
            - message
          properties:
            message:
              type: string
              maxLength: 1000
              description: Nội dung tin nhắn. Giới hạn tối đa là 1000 ký tự.
              example: "TEXT message"
            caption:
              type: string
              maxLength: 20
              description: Chú thích trên nút. Giới hạn tối đa là 20 ký tự.
              example: "Click Here"
            url_action:
              type: string
              format: uri
              pattern: "^https://.*" # Đảm bảo URL sử dụng HTTPS
              maxLength: 255
              description: URL tài nguyên để chuyển hướng khi nhấn nút. Giới hạn tối đa là 255 ký tự. Yêu cầu giao thức HTTPS.
              example: "https://gapit.com.vn"
    ViberTextTemplateMessage:
      allOf:
        - $ref: "#/components/schemas/ViberBaseContent"
        - type: object
          required: [message]
          properties:
            message:
              type: string
              maxLength: 1000
              description: Nội dung văn bản đã đăng ký dưới dạng mẫu với GAPIT. Giới hạn tối đa là 1000 ký tự.
              example: "TEXT_TEMPLATE message"
    ViberImageMessage:
      allOf:
        - $ref: "#/components/schemas/ViberBaseContent"
        - type: object
          required:
            - url_img
          properties:
            url_img:
              type: string
              format: uri
              maxLength: 255
              description: Địa chỉ URL trực tiếp đến hình ảnh, bao gồm phần mở rộng của hình ảnh. Giới hạn tối đa là 255 ký tự. Các phần mở rộng được khuyến nghị - PNG, JPEG, jpg.
              example: "https://example.com/img.png"
            message:
              type: string
              maxLength: 1000
              description: Nội dung văn bản. Giới hạn tối đa là 1000 ký tự.
              example: "IMG message"
            caption:
              type: string
              maxLength: 20
              description: Chú thích trên nút. Giới hạn tối đa là 20 ký tự.
              example: "Click Here"
            url_action:
              type: string
              format: uri
              maxLength: 255
              description: URL tài nguyên để chuyển hướng khi nhấn nút. Giới hạn tối đa là 255 ký tự. Giao thức khuyến nghị - HTTPS.
              example: "https://gapit.com.vn"
    ViberFileMessage:
      allOf:
        - $ref: "#/components/schemas/ViberBaseContent"
        - type: object
          required:
            - file_name
            - file_type
            - url_file
          properties:
            file_name:
              type: string
              maxLength: 255
              description: Tên của tệp tin sẽ được hiển thị trong tin nhắn. Giới hạn tối đa là 255 ký tự.
              example: "myFile.docx"
            file_type:
              type: string
              enum:
                - doc
                - docx
                - rtf
                - dot
                - dotx
                - odt
                - odf
                - fodt
                - txt
                - info
                - pdf
                - xps
                - pdax
                - eps
                - xls
                - xlsx
                - ods
                - fods
                - csv
                - xlsm
                - xltx
              description: |
                Định dạng tệp hỗ trợ. Chỉ chấp nhận các giá trị sau:
                - Documents: .doc, .docx, .rtf, .dot, .dotx, .odt, .odf, .fodt, .txt, .info
                - PDF: .pdf, .xps, .pdax, .eps
                - Spreadsheet: .xls, .xlsx, .ods, .fods, .csv, .xlsm, .xltx
              example: "docx"
            url_file:
              type: string
              format: uri
              maxLength: 255
              description: Địa chỉ URL trực tiếp đến tệp tin, bao gồm phần mở rộng tệp. Giới hạn tối đa là 255 ký tự. Giao thức khuyến nghị - HTTPS.
              example: "https://example.com/file.docx"
    ViberVideoMessage:
      allOf:
        - $ref: "#/components/schemas/ViberBaseContent"
        - type: object
          required:
            - url_video
            - url_thumbnail
            - file_size
            - duration
          properties:
            url_video:
              type: string
              format: uri
              maxLength: 255
              description: Địa chỉ URL trực tiếp đến video, bao gồm phần mở rộng video. Giới hạn tối đa là 255 ký tự. Giao thức khuyến nghị - HTTPS.
              example: "https://example.com/video.mp4"
            url_thumbnail:
              type: string
              format: uri
              maxLength: 255
              description: Địa chỉ URL đến ảnh thu nhỏ (thumbnail). Giới hạn tối đa là 255 ký tự. Giao thức khuyến nghị - HTTPS.
              example: "https://example.com/thumbnail.png"
            file_size:
              type: integer
              minimum: 1
              maximum: 200000000
              description: Kích thước video tính bằng byte. Phạm vi từ 1 đến 200,000,000 byte.
              example: 300000
            duration:
              type: integer
              minimum: 1
              maximum: 600
              description: Thời gian dài của video tính bằng giây. Phạm vi từ 1 đến 600 giây.
              example: 60
            message:
              type: string
              maxLength: 1000
              description: Nội dung văn bản. Giới hạn tối đa là 1000 ký tự.
              example: "VIDEO message"
            caption:
              type: string
              maxLength: 20
              description: Chú thích trên nút. Giới hạn tối đa là 20 ký tự.
              example: "Click Here"
            url_action:
              type: string
              format: uri
              maxLength: 255
              description: URL tài nguyên để chuyển hướng khi nhấn nút. Giới hạn tối đa là 255 ký tự. Giao thức khuyến nghị - HTTPS.
              example: "https://gapit.com.vn"
    ViberPredefinedTemplateMessage:
      allOf:
        - $ref: "#/components/schemas/ViberBaseContent"
        - type: object
          required:
            - template_id
            - template_params
          properties:
            template_id:
              type: string
              description: >
                Mã template đã được định nghĩa trước. Mỗi mã tương ứng với một loại tin nhắn cụ thể:
                  - `0aac888f-2ee2-4112-9659-1755a951966a`: Basic OTP
                    - English Copy: Your code is {{pin}}.
                    Please don't share your code with ANYONE. We'll never call or
                    message you and ask for it.
                    - Parameters and Validations: {{pin}} - TEXT
                  - `c2cdc028-a48b-4187-bbaf-3aaa137b6e23`: Basic OTP valid for 5 minutes
                    - English Copy: Your code is {{pin}}. It's valid for 5 minutes.
                        Please don't share your code with ANYONE. We'll never call or
                        message you and ask for it.
                    - Parameters and Validations: {{pin}} - TEXT
                  - `c33cccaf-735f-4d62-8383-a76bf4999e2d`: OTP with business/platform name
                    - English Copy: {{business_platform_name}}: Your code is {{pin}}.
                        Please don't share your code with ANYONE. We'll never call or
                        message you and ask for it.
                    - Parameters and Validations: {{pin}} - TEXT
                        {{business_platform_name}} - TEXT
                  - `bed0942f-e07e-4879-834b-29cc4cf3ec35`: Basic OTP with code validity time
                    - English Copy: Your code is {{pin}}. It's valid for {{code_validity_time}} minutes.
                        Please don't share your code with ANYONE. We'll never call or
                        message you and ask for it.
                    - Parameters and Validations: {{pin}} - TEXT
                        {{code_validity_time}} - NUMBER
                  - `60d67b38-b9fb-444a-80e6-754447e010c6`: OTP with code type & validity time
                    - English Copy: Your {{pin_type}} code is {{pin}}. It's valid for {{code_validity_time}}
                        minutes.
                        Please don't share your code with ANYONE. We'll never call or
                        message you and ask for it.
                    - Parameters and Validations: {{pin}} - TEXT
                        {{pin_type}} - TEXT
                        {{code_validity_time}} - NUMBER
                  - `6c929cef-29b4-4349-bc9d-2a07bdbb6e43`: OTP with business/platform name & validity time
                    - English Copy: Your {{business_platform_name}} code is {{pin}}. It's valid for
                        {{code_validity_time}} hours.
                        Please don't share your code with ANYONE. We'll never call or
                        message you and ask for it.
                    - Parameters and Validations: {{pin}} - TEXT
                        {{business_platform_name}} - TEXT
                        {{code_validity_time}} - NUMBER
                  - `82ba31a3-db42-4e87-82e8-33fa92b9e5ed`: OTP with type of code type
                    - English Copy: Your Your {{pin_type}} code is {{pin}}.
                        Please don't share your code with ANYONE. We'll never call or
                        message you and ask for it.
                    - Parameters and Validations: {{pin}} - TEXT
                        {{pin_type}} - TEXT
                  - `210ee8a9-1ed5-43cd-96e4-65bba311ab40`: OTP with details on OTP action/platform
                    - English Copy: Your one-time password for {{pin_type}} is {{pin}}.
                        Please don't share your code with ANYONE. We'll never call or
                        message you and ask for it.
                    - Parameters and Validations: {{pin}} - TEXT
                        {{pin_type}} - TEXT
                  - `be56d97b-2a33-4c89-ac5f-f555a2c827d9`: OTP with business/platform name & code reason
                    - English Copy: {{business_platform_name}}: Your code is {{pin}}. You got this code
                        because {{code_reason}}.
                        Please don't share your code with ANYONE. We'll never call or
                        message you and ask for it.
                    - Parameters and Validations: {{pin}} - TEXT
                        {{business_platform_name}} - TEXT
                        {{code_reason}} - TEXT
              enum:
                - "0aac888f-2ee2-4112-9659-1755a951966a"
                - "c2cdc028-a48b-4187-bbaf-3aaa137b6e23"
                - "c33cccaf-735f-4d62-8383-a76bf4999e2d"
                - "bed0942f-e07e-4879-834b-29cc4cf3ec35"
                - "60d67b38-b9fb-444a-80e6-754447e010c6"
                - "6c929cef-29b4-4349-bc9d-2a07bdbb6e43"
                - "82ba31a3-db42-4e87-82e8-33fa92b9e5ed"
                - "210ee8a9-1ed5-43cd-96e4-65bba311ab40"
                - "be56d97b-2a33-4c89-ac5f-f555a2c827d9"
              example: "0aac888f-2ee2-4112-9659-1755a951966a"
            template_params:
              type: object
              description: Các biến yêu cầu cho template. Có thể có giá trị trong Kho template.
            template_lang:
              type: string
              description: >
                Mã ngôn ngữ để xác định ngôn ngữ của tin nhắn. 
                Giá trị mặc định là ngôn ngữ của thiết bị. Các giá trị hợp lệ bao gồm:
                - `en`: English
                - `ar`: Arabic
                - `bg`: Bulgarian
                - `hr`: Croatian
                - `cs`: Czech
                - `da`: Danish
                - `de`: German
                - `el`: Greek
                - `es`: Spanish
                - `fi`: Finnish
                - `fr`: French
                - `he`: Hebrew
                - `my`: Burmese
                - `hu`: Hungarian
                - `id`: Indonesian
                - `it`: Italian
                - `ja`: Japanese
                - `nb`: Norwegian
                - `nl`: Dutch
                - `pl`: Polish
                - `pt`: Portuguese (Portugal)
                - `br`: Portuguese (Brazil)
                - `ro`: Romanian
                - `ru`: Russian
                - `sk`: Slovak
                - `sr`: Serbian
                - `sv`: Swedish
                - `th`: Thai
                - `tr`: Turkish
                - `uk`: Ukrainian
                - `vi`: Vietnamese
                - `fa`: Persian
                - `be`: Belarusian
              enum:
                - en
                - ar
                - bg
                - hr
                - cs
                - da
                - de
                - el
                - es
                - fi
                - fr
                - he
                - my
                - hu
                - id
                - it
                - ja
                - nb
                - nl
                - pl
                - pt
                - br
                - ro
                - ru
                - sk
                - sr
                - sv
                - th
                - tr
                - uk
                - vi
                - fa
                - be
              example: "vi"
    Email:
      title: Email
      type: object
      properties:
        channel:
          type: string
          enum:
            - email
          description: |
            email channel
        from:
          type: string
          format: email
          description: sender email
          example: info@gapone.vn
        to:
          type: array
          description: "recipient "
          items:
            type: string
            format: email
            example: customer@gmail.com
        cc:
          type: array
          description: "recipient "
          items:
            type: string
            format: email
            example: customer@gmail.com
        bcc:
          type: array
          description: "recipient "
          items:
            type: string
            format: email
            example: customer@gmail.com
        template_id:
          type: string
          example: "15"
          description: Gapone email template id
        unsub_redirect_url:
          type: string
          format: uri
          description: |
            email unsubscribe url
          example: "https://unsub.abc_back.com/email/558"
        subject:
          type: string
          example: Thong bao tai khoan
          description: email subject
        content:
          type: object
          description: |
            email body (click the property name for details)
          minProperties: 1
          maxProperties: 2
          properties:
            text:
              type: string
              description: email body in text format
              example: "Thong bao tai khoan: account 55875247525, balance 1000000 vnd, date 25/06/2020 11:20. Chi tiet: https://gap1.vn/612345"
            html:
              type: string
              description: email body in html format
              example: "<h5> Thong bao tai khoan</h5> account 55875247525, balance 1000000  vnd, date 25/06/2020 11:20. <h6>Chi tiet</h6> https://gap1.vn/612345 /h5>"
      required:
        - channel
        - from
        - to
        - subject
        - content
      x-examples:
        example-1:
          channel: email
          from: info@gapone.vn
          to:
            - customer@gmail.com
          cc:
            - customer@gmail.com
          bcc:
            - customer@gmail.com
          template_id: "15"
          unsub_redirect_url: "https://unsub.abc_back.com/email/558"
          subject: Thong bao tai khoan
          content:
            text: "Thong bao tai khoan: account 55875247525, balance 1000000 vnd, date 25/06/2020 11:20. Chi tiet: https://gap1.vn/612345"
            html: "<h5> Thong bao tai khoan</h5> account 55875247525, balance 1000000  vnd, date 25/06/2020 11:20. <h6>Chi tiet</h6> https://gap1.vn/612345 /h5>"
    Whatsapp:
      title: Whatsapp
      type: object
      x-examples:
        example-2:
          channel: whatsapp
          from: "84123456789"
          to: "84912345678"
          body: "Thong bao tai khoan: account 55875247525, balance 1000000 vnd, date  25/06/2020 11:20. Chi tiet: https://gap1.vn/612345"
      properties:
        channel:
          type: string
          enum:
            - whatsapp
          description: whatsapp channel
        from:
          type: string
          example: 84123456789
          description: Whatsapp sender number (Liên hệ Gapit để đăng ký), lấy danh sách sender number [tại đây](#tag/Brandname/paths/~1v1~1sender~1whatsapp/get)
        to:
          type: string
          example: "84912345678"
          description: recipient number
        template_id:
          type: string
          example: "xhahdsasad"
          description: mã template của mẫu tin đã đăng ký, lấy thông tin template đã đăng ký [tại đây](#tag/Brandname/paths/~1v1~1sender~1whatsapp~1{phone_number}~1templates/get)
        template_data:
          type: object
          description: giá trị các tham số trong mẫu tin
          example: { "param1": "value1", "param2": "value2" }
      required:
        - channel
        - from
        - to
        - template_id
    A2PMessage:
      title: A2PMessage
      type: object
      x-examples:
        example-1:
          request_id: 1312rfg3dgdgd
          message:
            channel: zalo
            from: "3123123131"
            to: "84912345678"
            template_id: "32311"
            template_data:
              title: Thông báo tài khoản
              account: "55875247525"
              balance: "1000000"
              date: "25/06/2020 11:20"
          fallback:
            - channel: sms
              from: Gapit
              to: "84912345678"
              content:
                text: " Thong bao tai khoan: account 55875247525, balance 1000000 vnd, date  25/06/2020 11:20. Chi tiet: https://gap1.vn/612345"
                msg_type: text
                sms_type: CSKH
      properties:
        request_id:
          type: string
          example: 1312rfg3dgdgd
          description: phải là duy nhất trong vòng 24h. Nếu gửi trùng request_id sẽ nhận lại được message_id của bản tin có request_id đầu tiên và việc gửi tin sẽ không được thực hiện
        message:
          oneOf:
            - $ref: "#/components/schemas/SMS"
            - $ref: "#/components/schemas/Zalo"
            - $ref: "#/components/schemas/Zalo_RSA"
            - $ref: "#/components/schemas/Zalo_Journey"
            - $ref: "#/components/schemas/ZaloOA"
            - $ref: "#/components/schemas/Viber"
            - $ref: "#/components/schemas/Email"
            - $ref: "#/components/schemas/Whatsapp"
          description: "a2p channel message ([**SMS**](#tag/SMS_Channel), [**ZNS**](#tag/Zalo_Channel), [**ZNS Mã hóa**](#tag/Zalo_RSA_Channel), [**ZNS hành trình**](#tag/Zalo_Journey_Channel), [**Viber**](#tag/Viber_Channel), [**Email**](#tag/Email_Channel), [**Whatsapp**](#tag/Whatsapp_Channel)"
        fallback:
          type: array
          minItems: 1
          maxItems: 3
          description: tự động chuyển đổi sang kênh dự phòng nếu kênh chính gửi thất bại
          items:
            anyOf:
              - $ref: "#/components/schemas/SMS"
              - $ref: "#/components/schemas/Zalo"
              - $ref: "#/components/schemas/Zalo_RSA"
              - $ref: "#/components/schemas/Zalo_Journey"
              - $ref: "#/components/schemas/ZaloOA"
              - $ref: "#/components/schemas/Viber"
              - $ref: "#/components/schemas/Email"
              - $ref: "#/components/schemas/Whatsapp"
      required:
        - request_id
        - message
    Contact:
      type: object
      description: contain at least Email or Phone
      x-examples:
        example-1:
          Email: khachuy@gapit.com.vn
          Phone: "84969533045"
          Address: Hà Nội
          LastName: Lê Mạnh
          FirstName: Hùng
          DateOfBirth: 15/06/1976
      minProperties: 1
      properties:
        id:
          type: integer
          minimum: 1
          description: contact id generate by Gapone
        Email:
          type: string
          format: email
        Phone:
          type: string
          description: vietnamese mobile phone number with country code
          example: "84936123456"
        Address:
          type: string
        LastName:
          type: string
        FirstName:
          type: string
        DateOfBirth:
          type: string
    ErrorResponse:
      title: ErrorResponse
      type: object
      properties:
        request_id:
          type: string
          example: 1312rfg3dgdgd
        error:
          type: string
          description: error details
          example: Invalid channel
      required:
        - request_id
        - error
      x-examples:
        example-1:
          request_id: 1312rfg3dgdgd
          error: Invalid channel
    SegmentFilter:
      title: SegmentFilter
      type: object
      description: Contact filter
      properties:
        operator:
          type: string
          enum:
            - Equal
            - NotEqual
            - Like
            - NotLike
            - StartWith
            - EndWith
            - Contain
            - GreaterThan
            - SmallerThan
            - Empty
            - NotEmpty
          example: Equal
          description: filter operator
        property:
          type: string
          description: contact property
          example: email
        value:
          type: string
          description: compare value
          example: test@gapone.vn
      required:
        - operator
        - property
      x-examples:
        example-1:
          operator: Equal
          property: email
          value: test@gapone.vn
    Segment:
      title: Segment
      type: object
      description: Contact Segment
      x-examples:
        example-1:
          id: 0
          name: VIP
          description: VIP user
          filters:
            - - operator: Equal
                property: email
                value: test@gapone.vn
      properties:
        id:
          type: integer
        name:
          type: string
          description: segment name
          example: VIP
        description:
          type: string
          example: VIP user
        filters:
          type: array
          description: list of filter. Filters in the same list in the inner layer represent "AND" logic
          minItems: 1
          uniqueItems: true
          items:
            type: array
            minItems: 1
            uniqueItems: true
            items:
              $ref: "#/components/schemas/SegmentFilter"
      required:
        - id
        - name
        - filters
    StatusQuery:
      title: StatusQuery
      type: object
      properties:
        message_id:
          type: array
          minItems: 1
          uniqueItems: true
          maxItems: 50
          description: gapone message id
          items:
            type: string
      required:
        - message_id
      x-examples:
        example-1:
          message_id:
            - 81f13950-19ca-11eb-94ac-35235df80632
            - 5545ea50-249c-5e4b-dadc-693dfa598051
    StatusQueryResponse:
      title: StatusQueryResponse
      type: object
      properties:
        message_id:
          type: string
          description: Gapone message id
        status(click for details):
          type: array
          description: list of message status
          items:
            type: object
            properties:
              channel:
                type: string
                enum:
                  - sms
                  - zalo
                  - viber
                  - email
                  - whatsapp
              sent_status:
                type: integer
                example: 0
                description: "sent_status = 0, success <br> sent_status ≠ 0, xảy ra lỗi trong quá trình gửi"
              sent_time:
                type: string
                format: date-time
                example: "2020-10-29T09:38:31.925Z"
              delivery_status:
                type: integer
                example: 0
                description: "sent_status >= 0, success <br>  sent_status ≠ 0, xảy ra lỗi trong quá trình nhận<br> Xem chi tiết trạng thái gửi tin [delivery_status](#tag/delivery_status)"
              delivery_time:
                type: string
                format: date-time
                example: "2020-10-29T09:38:32.683Z"
              received_time:
                type: string
                format: date-time
                example: "2021-10-29T10:00:59.120Z"
              error_status:
                type: integer
                example: -9
                description: "Trường này sẽ không được trả về nếu không xảy ra lỗi.<br> Nếu xảy ra lỗi trong quá trình gửi tin (send_status ≠ 0 ) thì giá trị của error_status là  send_status.<br> Nếu tin được gửi thành công nhưng xảy ra lỗi trong quá trình nhận thì giá trị của error_status là delivery_status.<br> Xem chi tiết error_status theo từng kênh [SMS](#section/SMS), [Viber](#section/Viber), [Zalo](#section/Zalo), [Email](#section/Email)."
              error_messsage:
                type: string
                example: Brandname is not registered
                description: "Trường này sẽ không được trả về nếu không xảy ra lỗi.<br> Xem chi tiết error_message theo từng kênh [SMS](#section/SMS), [Viber](#section/Viber), [Zalo](#section/Zalo), [Email](#section/Email)."
      required:
        - message_id
        - status
      x-examples:
        example-1:
          message_id: string
          status:
            - channel: sms
              sent_status: 0
              sent_time: "2020-10-29T09:38:31.925Z"
              delivery_status: 0
              delivery_time: "2020-10-29T09:38:32.683Z"
    Property:
      description: ""
      type: object
      properties:
        field_db:
          type: string
          minLength: 1
        name:
          type: string
          minLength: 1
        is_required:
          type: boolean
        default_value:
          type: string
        control_type:
          type: integer
          enum:
            - 1
            - 2
            - 3
            - 4
            - 5
          description: |-
            1-textbox, 2-datetime, 3-combobox,4-number,
            5-radio
      required:
        - field_db
        - name
        - is_required
        - control_type
      x-examples:
        example-1:
          field_db: LastName
          name: Họ
          is_required: true
          default_value: null
          control_type: 1
    EmailCode:
      title: Email
      type: object
      properties:
        "-1":
          description: Email is not verified.
    Product:
      description: Thông tin sản phẩm
      type: object
      x-examples:
        example-1:
          id: 1
          property:
            code: AONAM-1111
            name: Áo Nao
          variants:
            - sku: AONAM-1111-S
              price: 2542000
              title: Cỡ S
              quantity: 1
            - sku: AONAM-1111-M
              price: 2542000
              title: Cỡ M
              quantity: 1
      properties:
        id:
          type: number
          minimum: 1
          description: id của sản phẩm trên GAPONE
          readOnly: true
        property:
          type: object
          required:
            - code
            - name
          description: "Thuộc tính sản phẩm, xem thông tin thuộc tính tại https://app.gapone.vn/Property/Index"
          properties:
            code:
              type: string
              minLength: 1
              description: Mã duy nhất của sản phẩm
            name:
              type: string
              minLength: 1
              description: Tên sản phẩm
        variants:
          type: array
          items:
            $ref: "#/components/schemas/ProductVariant"
      required:
        - id
        - property
        - variants
    ProductVariant:
      description: Biến thể của sản phẩm
      type: object
      x-examples:
        example-1:
          sku: AONAM-1111-S
          price: 2542000
          title: Cỡ S
          quantity: 1
      properties:
        sku:
          type: string
          minLength: 1
          description: Mã sku duy nhất của biến thể
        price:
          type: integer
          minimum: 0
          description: Giá sản phẩm theo biến thể
        title:
          type: string
          minLength: 1
          description: Đặc tính của biến thể
        quantity:
          type: integer
          minimum: 0
          default: 0
          description: số lượng tồn kho của biến thể
      required:
        - sku
        - price
        - title
        - quantity
    LineItem:
      description: Thông tin sản phẩm trong đơn hàng
      type: object
      x-examples:
        example-1:
          sku: "15325675246"
          price: 22290000
          title: 160x300x34
          total: 22290000
          quantity: 1
          product_id: 361
          variant_id: 679
          product_name: Nệm Cao su
      properties:
        sku:
          type: string
          minLength: 1
          description: Mã sku duy nhất của biến thể
        price:
          type: number
          description: Giá bán
        title:
          type: string
          minLength: 1
          description: Mô tả biến thể
        total:
          type: number
          description: Tổng giá trị của sản phẩm
        quantity:
          type: number
          description: Số lượng sản phẩm
        product_id:
          type: number
          description: Id GAPONE của sản phẩm
        variant_id:
          type: number
          description: Id GAPONE của biến thể
        product_name:
          type: string
          minLength: 1
          description: Tên sản phẩm
      required:
        - sku
        - price
        - title
        - total
        - quantity
        - product_id
        - variant_id
        - product_name
    Deal:
      description: thông tin đơn hàng
      type: object
      x-examples:
        example-1:
          id: 1
          contact_id: 1
          property:
            total: 22290000
            discount: 0
            quantity: 1
            ordered_at: 1635766200
            total_order: 22290000
            order_number: Order12345
            order_source: lazada
            order_status: completed_payment
            transport_fee: 0
          line_items:
            - sku: "15325675246"
              price: 22290000
              title: 160x300x34
              total: 22290000
              quantity: 1
              product_id: "361"
              variant_id: 679
              product_name: Nệm AmericanStar Madrid
      properties:
        id:
          type: number
          description: id GAPONE của đơn hàng
          readOnly: true
        contact_id:
          type: number
          description: id GAPONE của khách hàng
        property:
          type: object
          required:
            - total
            - discount
            - quantity
            - ordered_at
            - total_order
            - order_number
            - order_source
            - order_status
            - transport_fee
          description: "thuộc tính đơn hàng, danh sách thuộc tính tại https://app.gapone.vn/Property/Index"
          properties:
            total:
              type: number
              description: Tổng giá trị các sản phẩm của đơn hàng
            discount:
              type: number
              description: giá trị giảm giá của đơn hàng
              default: 0
            quantity:
              type: number
              description: số sản phẩm xuất hiện trong line_items (riêng biệt theo sku)
            ordered_at:
              type: number
              description: thời gian đặt hàng theo unix timestamp
            total_order:
              type: number
              description: giá trị đơn hàng (số tiền cuối cùng khách hàng phải trả)
            order_number:
              type: string
              minLength: 1
              description: mã duy nhất của đơn hàng
            order_source:
              type: string
              minLength: 1
              description: nguồn đơn hàng
            order_status:
              description: |
                Trạng thái đơn hàng, [**danh sách tại đây**](#tag/order_status)
              type: string
            transport_fee:
              type: number
              default: 0
              description: phí vận chuyển
        line_items:
          type: array
          uniqueItems: true
          minItems: 1
          items:
            $ref: "#/components/schemas/LineItem"
      required:
        - id
        - contact_id
        - property
        - line_items
  securitySchemes:
    BasicAuth:
      type: http
      scheme: basic
      description: Cơ chế xác thực sử dụng HTTP Basic auhentication scheme, với username & password là các giá trị client_id & client_secret nhận được ở trên ([Tạo API keys](#section/Quy-trinh-ket-noi-dich-vu/Tao-API-keys-(client_id-va-client_secret):))
    OAuth2ClientCredsAuth:
      type: oauth2
      flows:
        clientCredentials:
          tokenUrl: "https://auth.gapone.vn/tokens"
          scopes:
            gateway: gateway
      description: |
        Auth-API trao đổi tham số qua HTTP request/response headers & body như ví dụ bên dưới.
        Cơ chế xác thực sử dụng HTTP Basic auhentication scheme, với username & password là các giá trị client_id & client_secret nhận được ở trên (mục 2. [QUY TRÌNH KẾT NỐI DỊCH VỤ](#section/Quy-trinh-ket-noi-dich-vu)).

        *Auth-API request (client_id=s6BhdRkqt3, client_secret=gX1fBat3bV):*

        ```
        POST /token HTTP/1.1
        Host: auth.gapone.vn
        Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW
        Content-Type: application/x-www-form-urlencoded
        Content-Length: 43

        grant_type=client_credentials&scope=gateway
        ```
        *Auth-API response (successful):*

        ```
        HTTP/1.1 200 OK
        Content-Type: application/json;charset=UTF-8
        Transfer-Encoding: chunked
        Cache-Control: no-store
        Pragma: no-cache

        {
        "access_token":"2YotnFZFEjr1zCsicMWpAA",
        "token_type":"bearer",
        "scope":"gateway",
        "expires_in":3600
        }
        ```
        *Auth-API response (unsuccessful): https://tools.ietf.org/html/rfc6749#section-5.2*

        ```

        HTTP/1.1 400 Bad Request
        Content-Type: application/json;charset=UTF-8
        Transfer-Encoding: chunked
        Cache-Control: no-store
        Pragma: no-cache
        {
        "error":"invalid_request"
        }
        ```

        *Curl example*

        ```curl
        curl --location --request POST 'https://auth.gapone.vn/token' \
        --header 'Authorization: Basic a2V5OnNlY3JldA==' \
        --header 'Content-Type: application/x-www-form-urlencoded' \
        --data-urlencode 'grant_type=client_credentials' \
        --data-urlencode 'scope=gateway'
        ```

        *NodeJS example*

        ```nodejs
        var axios = require('axios');
        var qs = require('qs');
        var data = qs.stringify({
         'grant_type': 'client_credentials',
        'scope': 'gateway' 
        });
        var config = {
          method: 'post',
          url: 'https://auth.gapone.vn/token',
          auth: {
          username: 'api_key',
          password: 'api_secret'
          },
          headers: { 
            'Content-Type': 'application/x-www-form-urlencoded'
          },
          data : data
        };

        axios(config)
        .then(function (response) {
          console.log(JSON.stringify(response.data));
        })
        .catch(function (error) {
          console.log(error);
        });

        ```
        *C# example*

        ```C#
        public class TokenModel
        {
            public string access_token { get; set; }
            public string token_type { get; set; }
            public string scope { get; set; }
            public int expires_in { get; set; }
        }
        /// <summary>
        /// 
        /// </summary>
        /// <param name="clientId" /*username*/ ></param>
        /// <param name="client_secret" /*password*/></param> 
        /// <returns></returns>
        public async Task<TokenModel> GetTokenGapone(string clientId, string client_secret)
        {
            TokenModel tokenModel = new TokenModel();
            var body = new Dictionary<string, string>
                    {
                        { "grant_type", "client_credentials" },
                        { "scope", "gateway" }
                    };

            string url = "https://auth.gapone.vn/token";
            using var client = new HttpClient();
            client.DefaultRequestHeaders.Clear();
            client.DefaultRequestHeaders.Authorization =
            new AuthenticationHeaderValue("Basic",Convert.ToBase64String(Encoding.ASCII.GetBytes($"{clientId}:{client_secret}")));
            client.DefaultRequestHeaders.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
            var data = new FormUrlEncodedContent(body);
           
            HttpResponseMessage response = await client.PostAsync(url, data);
            string responseBody = await response.Content.ReadAsStringAsync();
            if (response.IsSuccessStatusCode)
            {
              tokenModel = JsonConvert.DeserializeObject<TokenModel>(responseBody);
              return tokenModel;
            }else
            {

                throw new Exception($"GAPONE AUTH return unexpected response, status: {response.StatusCode}, body:{responseBody} ");

            }
         }
        ```

        *Mã lỗi: Auth-API server trả về mã lỗi qua HTTP response status.*

        Mã lỗi  | Mô tả | Cách xử lý
        ----------------|-------------|---------
        200 | Thành công |
        401 | Unauthorized | Kiểm tra tham số authentication (client_id & client_secret). Không retry.
        403 | Forbidden | Kiểm tra tham số đường dẫn và quyền truy cập. Không retry.
        429 | Too many request | Tốc độ gửi request quá cao. Cần giảm tốc độ gửi requests xuống mức cho phép (default 1 request/ second).
        400, 4xx | Bad Request | Kiểm tra tham số request. Không retry.
        503 | Server busy | Thử lại sau 5-10 giây.
        5xx | Lỗi server | Thử lại sau 5-10 giây. Liên hệ Customer Support.

security:
  - BasicAuth: []
  - OAuth2ClientCredsAuth: []
tags:
  - name: A2P Message
  - name: Contact
    description: |
      **Chú ý:** API đang trong quá trình phát triển và có thể thay đổi dựa trên kết quả thử nghiệm.

      API này có vai trò quản lý, tạo mới, sửa, xóa danh sách Contact.

      Cơ chế xác thực sử dụng HTTP Bearer auhentication scheme, với token là giá trị access_token
        nhận được từ Auth-API server ở [bước trên](#section/Authentication/OAuth2ClientCredsAuth) .
  - name: Segment
    description: |
      **Chú ý:** API đang trong quá trình phát triển và có thể thay đổi dựa trên kết quả thử nghiệm.

      API này có vai trò quản lý, tạo mới, sửa, xóa danh sách Segment.
  - name: dn_overview
    x-displayName: Mô tả chung
    description: |
      Trạng thái gửi tin nhắn tới handset (delivery status) được lưu trữ trong hệ thống GAPONE. GAPONE portal https://gapone.vn cung cấp đầy đủ trạng thái của từng tin nhắn, thống kê theo campaigns, tập khách hàng...

      Ngoài ra, đối tác có thể đăng ký nhận delivery status qua webhook đã khai báo trên portal.
  - name: dn_url
    x-displayName: Url
    description: |
      Webhook URL nhận status notification phải được khai báo trong mục Quản lý ứng dụng trên GAPONE portal, xem cách đăng ký [webhook tại đây](#section/Quy-trinh-ket-noi-dich-vu/DJang-ky-URL-(webhook)-nhan-delivery-notification:). Ví dụ:

      Webhook Url: https://webhook.gapone-partner.vn/deliver
  - name: dn_request
    x-displayName: Request
    description: |
      
         Bản tin delivery status được gửi qua HTTP/SSL, POST method, [xem request body theo vd](#tag/dn_example).  

         | Trường             | Kiểu dữ liệu | Mô tả                                                                 |  
         |--------------------|--------------|----------------------------------------------------------------------|  
         | request_id         | string       | ID duy nhất của yêu cầu                                              |  
         | campaign_id        | integer      | ID chiến dịch (nếu có)                                               |  
         | message_id         | string       | ID duy nhất của tin nhắn                                              |  
         | delivery_channel   | string       | Kênh gửi tin nhắn (ví dụ: `sms`, `viber`, `zalo`)                     |  
         | delivery_status    | integer      | [Trạng thái gửi tin nhắn](#tag/delivery_status) (ví dụ: `0` là thành công)                    |  
         | delivery_description | string    | Mô tả trạng thái gửi tin nhắn                                         |  
         | timestamp          | string       | Thời gian ghi nhận trạng thái (ISO 8601)                              |  
         | sent_time          | string       | Thời gian gửi tin nhắn sang nhà mạng                                                |  
         | telco              | string       | Nhà mạng của người nhận (kênh sms)                                               |  
         | delivery_time      | string       | Thời gian người dùng ghi nhận trạng thái (nhận, xem, click)                                |  
         | error_status       | string       | Mã lỗi (nếu có)                                                       |  
         | error_message      | string       | Mô tả lỗi (nếu có)                                                    |  

         Cơ chế xác thực sử dụng chữ ký qua request header X-GapOne-Signature. Signature được tính theo công thức:  

         X-GapOne-Signature = sha256(client_id + message_id + delivery_channel + timestamp)  
         Trong đó:  
           - sha256: thuật toán băm  
           - client_id: đã nhận ở mục 2.THỦ TỤC KẾT NỐI API  
           - message_id, channel, timestamp: giá trị các trường trong bản tin delivery notification
  - name: dn_response
    x-displayName: Response
    description: |-
      Nếu Webhook nhận request thành công, response sẽ được trả lại với giá trị HTTP status 200. Trường hợp nhận không thành công, webhook trả lại response với giá trị HTTP status 4xx, 5xx như mô tả dưới đây.

      <table> <thead> <tr> <th>Mã lỗi</th> <th>Mô tả</th> <th>Cách xử lý</th> </tr> </thead> <tbody><tr> <td>200</td> <td>Thành công </td><td rowspan="7"> GAPONE sẽ thử lại sau 5-10 giây, có thể lâu hơn. Lỗi kéo dài quá 24h có thể dẫn tớiGAPONE ngừng gửi delivery notification. Khi đó, hãy liên hệ Technical Support của chúng tôi để được hỗ trợ.</td> </tr> <tr> <td>401</td> <td>Unauthorized</td> </tr> <tr> <td>403</td> <td>Forbidden</td> </tr> <tr> <td>429</td> <td>Too many request</td> </tr> <tr> <td>400, 4xx</td> <td>Bad Request</td> </tr> <tr> <td>503</td> <td>Server busy</td> </tr> <tr> <td>5xx</td> <td>Lỗi server</td> </tr> </tbody></table>
  - name: dn_status
    x-displayName: Trạng thái gửi tin
    description: |
      Xem bảng mã trạng thái gửi tin (delivery_status) tới khách hàng [**tại đây**](#tag/delivery_status).
  - name: dn_example
    x-displayName: Ví dụ
    description: |
      *GAPONE Request:*
        ```
        POST /deliver HTTP/1.1
        Host: webhook.gapone-partner.vn
        X-GapOne-Signature: c6b37be3a85936b208d3051234ee236394520e324b45f3eb184e49deba2a06f2
        Content-Type: application/json;charset=UTF-8
         {
          "request_id": "b72b3cd58",
          "campaign_id": 0,
          "message_id": "a7b3cdf8-29e5-12g0-b2bd-425448e3762e",
          "delivery_channel": "sms",
          "delivery_status": 0,
          "delivery_description": "MESSAGE_DELIVERED",
          "timestamp": "2025-04-01T17:07:48.000+0700",
          "sent_time": "2025-04-01T17:07:47.000+0700",
          "telco": "viettel",
          "delivery_time": "2025-04-01T17:07:47.000+0700",
          "error_status": "200",
          "error_message": "Success"
          }
        ```
        *Webhook response (successful):*
        ```
        HTTP/1.1 200 OK
        Content-Type: application/json;charset=UTF-8
        Transfer-Encoding: chunked
        {
        "status": 200,
        "description": "delivery notification received",
        "request_id":"deliv-1a23c434-noti-165652df"
        }
        ```
        *Webhook response (unsuccessful):*
        ```
        HTTP/1.1 401 Unauthorized
        Content-Type: application/json;charset=UTF-8
        Transfer-Encoding: chunked
        {
        "status": 401,
        "description": "invalid signature",
        "request_id":"deliv-1a23c434-noti-165652df"
        }
        ```
  - name: error_status
    description: |
      # SMS

       Mã   | Mô tả | 
        ----------------|-------------|
        404 | Undefined Error |
        408 | Timeout | 
        -1 | Bad Request | 
        -2 | Invalid Length | 
        -3 | Message Advertise | 
        -4 | Phone Number In Blacklist | 
        -5 | Spam Message | 
        -6 | Template is not register |
        -8 | Invalid CpId, accout |
        -9 | Brandname is not registered |
        -10 | Invalid Source Ip |
        -15 | PhoneNumber Not Support |
        -18 | Message contains vietnamese sign (Viettel bank) |
        -19 | Message is not decrypted (viettel bank) |
        -32 | Out of quota |
        -999 | Server Error |

      # Viber

       Mã   | Mô tả | 
        ----------------|-------------|
        36013 | Viber internal failure |
        36023 | Viber invalid service id   |
        36033 | Viber invalid data |
        36043 | Viber blocked message type |
        36053 | Viber bad message type |
        36063 | Viber bad parameters |
        36073 | Viber timeout |
        36083 | Viber user blocked |
        36093 | Viber not viber user |
        36103 | Viber no suitable device |
        36113 | Viber unauthorized ip |
        36123 | Viber already sent |
        36133 | Viber not permitted |
        36143 | Viber billing failure |
        36153 | Viber word in black list |
        36163 | Viber internal process error |
        36173 | Viber wrong or missing Label |
        36183 | Viber invalid TTL value |
        35015 | Message has not been delivered within TTL |

      # Zalo

       Mã   | Mô tả | 
        ----------------|-------------|
        -100 | Phát sinh lỗi không xác định |
        -101 | Ứng dụng gửi ZNS không hợp lệ |
        -102 | Ứng dụng gửi ZNS không tồn tại |
        -103 | Ứng dụng gửi ZNS chưa được phê duyệt |
        -104 | Mã secret key không tồn tại |
        -105 | Ứng dụng gửi ZNS chưa liên kết với OA |
        -106 | Phương thức không được hỗ trợ |
        -107 | ID ZNS không hợp lệ |
        -108 | Số điện thoại không hợp lệ |
        -109 | ID mẫu ZNS không hợp lệ |
        -110 | Người dùng chưa cập nhật Zalo phiên bản mới |
        -111 | Mẫu ZNS không có dữ liệu |
        -112 | Dữ liệu mẫu ZNS không hợp lệ |
        -113 | Nút không hợp lệ |
        -114 | Người dùng không nhận được ZNS vì các lý do: Trạng thái tài khoản, Tùy chọn nhận ZNS, Sử dụng Zalo phiên bản cũ, hoặc các lỗi nội bộ khác |
        -115 | Ví không đủ số dư |
        -116 | Nội dung không hợp lệ |
        -117 | OA hoặc ứng dụng gửi ZNS chưa được cấp quyền sử dụng mẫu ZNS này |
        -118 | Số điện thoại chưa đăng ký Zalo hoặc người dùng đã vô hiệu hóa tài khoản hơn 30 ngày |
        -119 | Tài khoản không thể nhận ZNS |
        -120 | OA chưa được cấp quyền sử dụng tính năng này |
        -121 | Mẫu ZNS không có nội dung |
        -122 | Nội dung mẫu ZNS không đúng định dạng json |
        -123 | Không giải mã được nội dung mẫu ZNS |
        -124 | Mã truy cập không hợp lệ |
        -125 | ID OA không hợp lệ |
        -126 | Ví (development mode) không đủ số dư |
        -127 | Template test chỉ có thể được gửi cho admin |
        -128 | Ứng dụng gửi ZNS chưa có mã encoding |
        -129 | Mã encoding không thể được tạo ra |
        -130 | Nội dung mẫu ZNS vượt giới hạn kí tự |
        -131 | Mẫu ZNS chưa được phê duyệt |
        -132 | Tham số không hợp lệ |
        -133 | Không hỗ trợ gửi ZNS trong khung giờ 22h - 6h |
        -134 | Người dùng chưa phản hồi gợi ý nhận ZNS từ OA |
        -135 | OA chưa được cấp quyền gửi ZNS |
        -136 | Ứng dụng gửi ZNS cần được kết nối với ZBA để sử dụng các tính năng trả phí |
        -137 | ZBA kết nối với ứng dụng gửi ZNS này đã hết số dư tài khoản hoặc không thể thanh toán |
        -138 | Ứng dụng gửi ZNS chưa có quyền sử dụng tính năng này |
        -139 | Người dùng đã từ chối nhận loại ZNS này |
        -140 | OA chưa được cấp quyền gửi ZNS hậu mãi cho người dùng này |
        -141 | Người dùng từ chối nhận ZNS từ OA |
        -142 | OA chưa có RSA key, vui lòng gọi API tạo RSA key |
        -143 | OA đã có RSA key, vui lòng gọi API get RSA key |
        -144 | OA đã vượt quá số lượng gửi cho phép trong ngày hôm nay |
        -145 | OA không được phép gửi loại nội dung ZNS này |
        -146 | Mẫu ZNS này đã bị vô hiệu hóa do chất lượng thấp |
        -147 | OA đã vượt quá số lượng gửi cho phép của mẫu ZNS này trong ngày hôm nay |

      # Whatsapp

       Mã   | Mô tả | 
        ----------------|-------------|
        20003 | Permission Denied |
        20500 | Internal Server Error |
        21211 | Invalid 'To' Phone Number |
        21212 | Invalid From Number (caller ID)
        30001 | Queue overflow  |
        30008 | Unknown error |
        63003 | Channel could not find To address |
        63005 | Channel did not accept given content  |
        63007 | Could not find a Channel with the From address  |
        63013 | Channel policy violation  |
        63016 | Failed to send freeform message because you are outside the allowed window  |
        63012 | Channel returned an internal service error |
        63021 | Channel invalid content error   |
        63031 | Channels message cannot have same From and To |

      # Email

        Mã   | Mô tả | 
        ----------------|-------------|
        -1 | Email is not verified |
  - name: delivery_status
    description: |
      <span style="color:red"> Chú ý: Đối với các tin có delivery_status là -4,-5,-6 nên gọi lại API [Lấy trạng thái gửi tin](#operation/post-status-query) vào đầu của tháng kế tiếp trước khi đối soát</span>.

      Mã | Mô tả
      ----------------|-------------
      -6 | OFFLINE_TIMEOUT (ZNS OTP only): người dùng chưa nhận được tin nhắn sau 15 giây từ thời điểm gửi sang Zalo (sẽ cập nhật lại =0 nếu người dùng nhận được tin nhắn)
      -5 | OFFLINE_TIMEOUT (ZNS only): người dùng chưa nhận được tin nhắn sau 120 phút từ thời điểm gửi sang Zalo (sẽ cập nhật lại =0 nếu người dùng nhận được tin nhắn)
      -4 | OFFLINE_TIMEOUT (ZNS only): người dùng chưa nhận được tin nhắn sau 10 phút từ thời điểm gửi sang Zalo (sẽ cập nhật lại =0 nếu người dùng nhận được tin nhắn)
      -3 | message_delayed (Email only)
      -2 | message_bounced (Email only)
      -1 | message_undelivered
       0 | message_delivered: người dùng nhận được tin nhắn
       1 | message_sent
       2 | message_opened: người dùng xem(mở) tin nhắn
       3 | message_clicked (Email only)
       4 | message_complaine (Email only)
  - name: order_status
    description: |
      Mã | Mô tả
      ----------------|-------------
      placed_order | Đơn hàng mới
      confirming_order | Đơn xác nhận
      confirmed_order | Đã xác nhận
      completed_payment | Đã thanh toán
      packing_products | Đang đóng gói sản phẩm
      transfering_products | Đổi kho xuất hàng
      waiting_for_pickup | Chờ lấy hàng
      picking_up | Đang lấy hàng
      picked_up | Đã lấy hàng
      fulfilling_order | Đang vận chuyển
      completed_order | Hoàn thành
      unsuccessful_fulfillment | Giao không thành công
      returning_fulfillment | Chờ chuyển hoàn
      returned_fulfillment | Đã chuyển hoàn
      cancelled_order | Đã hủy
      refunded_order | Đã hoàn tiền
  - name: FAQs
    description: |
      1. Q: Nếu kênh trong request chính thất bại, nhưng kênh Fallback gửi thành công thì API check status sẽ nhận kết quả detail kênh nào thành công, kênh nào thất bại?<br>
         A: Chi tiết cụ thể có thể kiểm tra qua API Status-query.

      2. Q: Trường hợp nào Gapone sẽ sử dụng đẩy tin theo Fallback? (VD: kênh chính có kết quả thất bại? Template request kênh chính không đúng yêu cầu?)<br>
         A: Request gửi sang đúng format => kênh chính gửi kết quả thất bại tường minh (vd:gửi SMS sai brandname,sai template..) => đẩy tin theo fallback

      3. Q: Khi API gửi tin thất bại, Gapone có thể bổ sung lý do cụ thể tại sao thất bại không? (VD: SĐT KH không có Zalo)<br>
         A: Có thể hiển thị đầy đủ qua API Status-query các mã lỗi cụ thể của các kênh sms,zalo,viber trong tài liệu.
         
      4. Q: Sau khi Zalo duyệt, bên khách hàng sẽ cần Template ID Zalo để định danh mẫu tin trong API send, ID này sẽ lấy ở đâu?<br>
         A: Chờ Gapone gửi manual lại (hiện tại chưa có cách lấy trên portal).<br>
            Thời gian chờ sẽ theo lịch làm việc trong tuần.<br>
            Trường hợp cần đổi thông tin (VD Logo) thì phải đăng ký mẫu mới vì Template đã duyệt thì ko được sửa.
         
      5. Q: Gapone có cung cấp API get đánh giá của Viber/Zalo về các tin nhắn KH nhận được không?<br>
         A: Việc đánh giá đang được Zalo quản trị nên Gapone không có API.
               
      6. Q: Thời gian nhận được delivery từ các kênh zalo, viber, sms ? <br>
         A: Khoảng vài giây, với viber hệ thống đang nâng cấp , nên delivery trả sẽ lâu hơn
         
      7. Q: Webhook có gửi kết quả thất bại không ? <br>
         A: Có.
  - name: SMS_Channel
    x-displayName: SMS Message
    description: |
      <SchemaDefinition schemaRef="#/components/schemas/SMS" />
  - name: Viber_Channel
    x-displayName: Viber Message
    description: |
      <SchemaDefinition schemaRef="#/components/schemas/Viber" />
  - name: Email_Channel
    x-displayName: Email Message
    description: |
      <SchemaDefinition schemaRef="#/components/schemas/Email" />
  - name: Zalo_Channel
    x-displayName: ZNS
    description: |
      <SchemaDefinition schemaRef="#/components/schemas/Zalo" />
  - name: Zalo_RSA_Channel
    x-displayName: ZNS Mã Hóa
    description: |
      Trước khi gửi tin cần [Tạo RSA public key tại đây](#tag/Zalo/paths/~1v1~1zns~1rsa/post).

      Mã hóa giá trị trong template_data, số điện thoại (to) theo code sau (Nodejs):
      ```javascript
        const dataToEncrypt = '123456';
        const utf8 = require('utf8');

        const publicKeySring = 'publickey from zns rsa key gen api';

        const publicKeyPem = `-----BEGIN PUBLIC KEY-----\n${publicKeyS}\n` + '-----END PUBLIC KEY-----';

        const forge = require('node-forge');

        const { rsa } = forge.pki;

        const publicKey = forge.pki.publicKeyFromPem(publicKeyPem);

        const encrypted = publicKey.encrypt(utf8.encode(dataToEncrypt), 'RSA-OAEP',{
          md: forge.md.sha256.create(),
          mgf1: {
            md: forge.md.sha1.create()
          }
        });
        console.log('encrypted:', forge.util.encode64(encrypted));
      ```
      <SchemaDefinition schemaRef="#/components/schemas/Zalo_RSA" />
  - name: Zalo_Journey_Channel
    x-displayName: ZNS Hành Trình
    description: |
      Trước khi gửi tin cần [Tạo journey token tại đây](#tag/Zalo/paths/~1v1~1zns~1{oa_id}~1journey~1{phone}?token_type={token_type}/post).

      *[Tìm hiểu thêm thông tin tại đây](https://developers.zalo.me/docs/zalo-notification-service/gui-tin-zns/gui-zns-journey)*
      <SchemaDefinition schemaRef="#/components/schemas/Zalo_Journey" />
  - name: Zalo_OA_Channel
    x-displayName: Zalo OA Message
    description: |
      <SchemaDefinition schemaRef="#/components/schemas/ZaloOA" />
  - name: Whatsapp_Channel
    x-displayName: Whatsapp Message
    description: |
      <SchemaDefinition schemaRef="#/components/schemas/Whatsapp" />
  - name: Tracking_Introduction
    x-displayName: Giới thiệu
    description: |
      Web tracking code sử dụng áp dụng cho các khách hàng sử dụng website tự tạo hoặc được tạo từ các ứng dụng WordPress/Woocommerce, Shopify,...

      * **GAPONE cung cấp 2 nhóm sự kiện website:**
        * Sự kiện mặc định của website
        * Sự kiện tuỳ chỉnh theo nhu cầu (Custom events)

      * **Quy trình kích hoạt sự kiện mặc định - website tracking:**
        * Bước 1: Cài đặt client site tracking code, xem [**tại đây**](#tag/Tracking_Install).
        * Bước 2: Khai báo Gapone Website Object, xem [**tại đây**](#tag/Gapone_Object).
        * Bước 3: Thiết lập theo dõi sự kiện mặc định theo mẫu của GAPONE, xem tại [**tại đây**](#tag/Tracking_Event).
        * Bước 4 (nếu có nhu cầu): Thiết lập theo dõi sự kiện website tuỳ chỉnh Server site API như hướng dẫn [**tại đây**](#tag/Tracking_Event_Server).

      * **Với trường hợp khách hàng sử dụng website Haravan có nhu cầu kết nối thêm sự kiện, quy trình kích hoạt như dưới đây:**
        * Cài đặt client site tracking code, xem [**tại đây**](#tag/Tracking_Install).
  - name: Gapone_Object
    x-displayName: Khai báo Gapone Website Object
    description: |
      Gapone Object là một đối tượng JavaScript (Javascript Object) cho phép GAPONE thu thập thông tin người dùng trong các sự kiện mặc định trên [**website**](#tag/Default_Event_Types). Gapone object phải được khai báo trước phần [**tracking code**](#tag/Tracking_Install).
      Khai báo như sau (javascript):

      ```JavaScript
        window.gapone_object = window.gapone_object || {};

      ```
      Dưới đây là danh sách các đối tượng thuộc gapone object:
      ## Page
      Page Object chứa các thuộc tính của trang hiện tại

      * Danh sách thuộc tính

          Thuộc tính | Kiểu dữ liệu | Mô tả
          -----------|-------------|-------
          type | string | Loại trang. Các loại trang mặc định bao gồm: home(Trang chủ), product(Trang chi tiết sản phẩm), cart(Trang giỏ hàng), checkout(Trang thanh toán đơn hàng), catetogry(Trang danh mục sản phẩm theo loại)
          title | string | Tên trang 
          is_log_in | boolean | Trạng thái đăng nhập của người dùng
      * Page Object Sample
      ```
          window.gapone_object = window.gapone_object || {};
          gapone_object.page = {
              "type": "product",
              "title": "Sản phẩm A",
              "is_log_in": true,
          }
      ```
        ## User
      User Object chứa các thông tin của người truy cập trang

      * Danh sách thuộc tính

          Thuộc tính | Kiểu dữ liệu | Mô tả
          -----------|-------------|-------
          email | string | Email
          phone | string | Số điện thoại
      * User Object Sample
      ```
          window.gapone_object = window.gapone_object || {};
          gapone_object.user = {
              "email": "abc123@gmail.com",
              "phone": "0968523984",
          }
      ```


      ## Product
      Product Object chứa các thuộc tính của sản phẩm

      * Danh sách thuộc tính

          Thuộc tính | Kiểu dữ liệu | Mô tả
          -----------|-------------|-------
          product_name | string | Tên sản phẩm
          product_code | string | Mã duy nhất của sản phẩm
          category | string | Loại hàng hóa
          sku | string | Mã sku duy nhất của biến thể
          title | string | Đặc tính của biến thể
          price | integer | Giá sản phẩm theo biến thể

      * Product Object Sample
      ```
          window.gapone_object = window.gapone_object || {};
          gapone_object.product = {
              "product_name": "Đầm Sát Nách Phối Bèo",
              "product_code": "1030971764",
              "product_category": "Váy Nữ",
              "sku": "haravan-1067924393",
              "title": "Đầm Sát Nách Phối Bèo - S - Xanh",
              "price": 1200000
          }
      ```
      ## Cart
      Cart Object chứa các thuộc tính của sản phẩm trong giỏ hàng

      * Danh sách thuộc tính
          Thuộc tính | Kiểu dữ liệu | Mô tả
          -----------|-------------|-------
          quantity | integer | Tổng số lượng sản phẩm trong giỏ hàng. Ví dụ: Có 3 sản phẩm A, 2 sản phẩm B -> Tổng số lượng sản phẩm = 5
          total_order | integer | Tổng giá trị giỏ hàng   
          line_items | array | Danh sách sản phẩm trong giỏ hàng

      * Cart Object Sample
      ```
          window.gapone_object = window.gapone_object || {};
          gapone_object.cart = {
              "quantity": 1,
              "total_order": 1700000,
              "line_items": [
              {"product_code":"1030971768",
              "sku":"1067924402",
              "quantity":1,
              "title":"S / h / h",
              "product_name":"Đầm Hai Dây In Hoa Với Thiết Kế Cổ Chữ V",
              "price":12000000,
              "total":12000000}],
          }
      ```
  - name: Default_Event_Types
    x-displayName: Danh sách các sự kiện mặc định
    description: |
      GAPONE định nghĩa danh sách sự kiện mặc định website như sau:

        Số thứ tự | Tên sự kiện | Mô tả
        -----------|-------------|-------
        1 | Xem trang | Khách hàng truy cập vào trang web bất kỳ
        2 | Thoát trang | Khách hàng thoát trang web bất kỳ
        3 | Đăng ký tài khoản | Khách hàng đăng ký tài khoản thành công
        4 | Đăng nhập | Khách hàng đăng nhập thành công
        5 | Thêm sản phẩm vào giỏ | Khách hàng thêm sản phẩm vào giỏ hàng thành công
        6 | Xoá sản phẩm khỏi giỏ | Khách hàng xoá sản phẩm khỏi giỏ hàng thành công
        7 | Mua hàng thành công | Khách hàng mua hàng thành công, khai báo cho sự kiện 9 - Bỏ quên giỏ hàng
        8 | Mua hàng không thành công | Khách hàng mua hàng không thành công (Người dùng đã xác nhận thông tin giao / nhận hàng, nhưng thực hiện thanh toán không thành công)
        9 | Bỏ quên giỏ hàng | Định nghĩa theo Sự kiện Thêm sản phẩm vào giỏ & Không có sự kiện Mua hàng thành công
        10 | Xem sản phẩm nhưng không thêm vào giỏ | Định nghĩa theo Sự kiện Xem trang & Không có sự kiện Thêm sản phẩm vào giỏ hàng


      ## Sự kiện xem trang: gapone_web_page_view

      * Sự kiện xảy ra khi khách hàng truy cập vào trang web bất kỳ, sự kiện được tự động thu thập bởi Gapone thông qua việc khai báo [**gapone object**](#tag/Gapone_Object)
      * Danh sách thuộc tính (tự động thu thập qua [**gapone object**](#tag/Gapone_Object))
          Thuộc tính | Kiểu dữ liệu | Mô tả
          -----------|-------------|-------
          page_type | string | Loại trang (bắt buộc)
          cart_quantity | integer | Tổng số lượng sản phẩm trong giỏ hàng. Ví dụ: Có 3 sản phẩm A, 2 sản phẩm B -> Tổng số lượng sản phẩm = 5   
          cart_total_order | integer | Tổng giá trị sản phẩm trong giỏ hàng
          is_log_in | boolean | Người dùng có đăng nhập hay không
          category_name| string | Tên danh mục sản phẩm nếu page_type='category'
          product_sku| string | Sku của sản phẩm nếu page_type='product'
          product_code| string | Mã của sản phẩm nếu page_type='product'
          product_name| string | Tên của sản phẩm nếu page_type='product'
          product_price| integer | Giá của sản phẩm nếu page_type='product'
          product_category| string | Phân loại sản phẩm nếu page_type='product'
          product_variant_title| string | Tên biến thể của sản phẩm nếu page_type='product'

      ## Sự kiện thoát trang: gapone_web_user_close_page

      * Sự kiện xảy ra khi khách hàng thoát trang web bất kỳ, sự kiện được tự động thu thập bởi Gapone thông qua việc khai báo [**gapone object**](#tag/Gapone_Object)
      * Danh sách thuộc tính (tự động thu thập qua [**gapone object**](#tag/Gapone_Object))
          Thuộc tính | Kiểu dữ liệu | Mô tả
          -----------|-------------|-------
          page_type | string | Loại trang (bắt buộc)
          cart_quantity | integer | Tổng số lượng sản phẩm trong giỏ hàng. Ví dụ: Có 3 sản phẩm A, 2 sản phẩm B -> Tổng số lượng sản phẩm = 5   
          cart_total_order | integer | Tổng giá trị sản phẩm trong giỏ hàng
          is_log_in | boolean | Người dùng có đăng nhập hay không
          category_name| string | Tên danh mục sản phẩm nếu page_type='category'
          product_sku| string | Sku của sản phẩm nếu page_type='product'
          product_code| string | Mã của sản phẩm nếu page_type='product'
          product_name| string | Tên của sản phẩm nếu page_type='product'
          product_price| integer | Giá của sản phẩm nếu page_type='product'
          product_category| string | Phân loại sản phẩm nếu page_type='product'
          product_variant_title| string | Tên biến thể của sản phẩm nếu page_type='product'

      ## Sự kiện đăng ký tài khoản: gapone_web_user_signup

      * Sự kiện xảy ra khi khách hàng đăng ký tài khoản thành công
      * Ví dụ:
      ```javascript
          window.gapone_object = window.gapone_object || {};
          gapone_object.user = {
              "email": "abc123@gmail.com",
              "phone": "0968523984",
          }
          GaponeScript.trackEvent('gapone_web_user_signup',{});
      ```

      ## Sự kiện đăng nhập: gapone_web_user_login

      * Sự kiện xảy ra khi khách hàng đăng nhập thành công
      * Ví dụ:
      ```javascript
          window.gapone_object = window.gapone_object || {};
          gapone_object.user = {
              "email": "abc123@gmail.com",
              "phone": "0968523984",
          }
          GaponeScript.trackEvent('gapone_web_user_login',{});
      ```

      ## Sự kiện thêm sản phẩm vào giỏ: gapone_web_cart_add

      * Sự kiện xảy ra khi khách hàng thêm sản phẩm vào giỏ hàng thành công
      * Danh sách thuộc tính 
          Thuộc tính | Kiểu dữ liệu | Mô tả
          -----------|-------------|-------
          sku| string | Sku của sản phẩm
          product_code| string | Mã của sản phẩm  
          product_name| string | Tên của sản phẩm  
          price| integer | Giá của sản phẩm
          quantity| integer | Số lượng sản phẩm thêm vào
          product_category| string | Phân loại sản phẩm  
          title| string | Tên biến thể của sản phẩm
          total| integer | Giá trị sản phẩm thêm vào
          cart_quantity| integer | Tổng số lượng sản phẩm trong giỏ hàng. Ví dụ: Có 3 sản phẩm A, 2 sản phẩm B -> Tổng số lượng sản phẩm = 5
          cart_total_order| integer | Tổng giá trị sản phẩm trong giỏ

      * Ví dụ:
      ```javascript
         GaponeScript.trackEvent('gapone_web_cart_add',{
        "product_code": "1030971764",
        "sku": "1067924393",
        "quantity": 1,
        "title": "Đầm Sát Nách Phối Bèo",
        "product_name": "Đầm Sát Nách Phối Bèo",
        "price": 12000000,
        "total": 12000000,
        "cart_quantity": 2,
        "cart_total_order": 24000000
      });
      ```

      ## Sự kiện xóa sản phẩm khỏi giỏ: gapone_web_cart_remove

      * Sự kiện xảy ra khi khách hàng xoá sản phẩm khỏi giỏ hàng thành công
      * Danh sách thuộc tính 
          Thuộc tính | Kiểu dữ liệu | Mô tả
          -----------|-------------|-------
          sku| string | Sku của sản phẩm
          product_code| string | Mã của sản phẩm  
          product_name| string | Tên của sản phẩm  
          price| integer | Giá của sản phẩm
          quantity| integer | Số lượng sản phẩm xóa
          product_category| string | Phân loại sản phẩm  
          title| string | Tên biến thể của sản phẩm
          total| integer |Tổng Giá trị sản phẩm xóa

      * Ví dụ:
      ```javascript
         GaponeScript.trackEvent('gapone_web_cart_remove',
        {"product_code":"1030971764",
        "sku":"1067924393",
        "quantity":1,
        "title":"S / mmm",
        "product_name":"Đầm Sát Nách Phối Bèo",
        "price":12000000,
        "total":12000000}
      );
      ```

      ## Sự kiện mua hàng thành công: gapone_web_user_purchase_successful

      * Sự kiện xảy ra khi khách hàng mua hàng thành công, khai báo cho sự kiện 9 - Bỏ quên giỏ hàng
      * Ví dụ:
      ```javascript
          window.gapone_object = window.gapone_object || {};
          gapone_object.user = {
              "email": "abc123@gmail.com",
              "phone": "0968523984",
          }
          GaponeScript.trackEvent('gapone_web_user_purchase_successful',{});
      ```
      ## Sự kiện mua hàng không thành công: gapone_web_purchase_unsuccessful

      * Sự kiện xảy ra khi khách hàng mua hàng không thành công (Người dùng đã xác nhận thông tin giao / nhận hàng, nhưng thực hiện thanh toán không thành công).
      * Ví dụ:
      ```javascript
          window.gapone_object = window.gapone_object || {};
          gapone_object.user = {
              "email": "abc123@gmail.com",
              "phone": "0968523984",
          }
          GaponeScript.trackEvent('gapone_web_purchase_unsuccessful',{});
      ```
  - name: Tracking_Install
    x-displayName: Cài đặt client site tracking code
    description: |
      Tạo kết nối website và lấy code cài đặt [**tại đây**](https://app.gapone.vn/EventTracking/Index)

      *Ghi chú:* Với khách hàng sử dụng website của Haravan, bộ sự kiện mặc định sẽ được áp dụng tự động ngay sau khi website được kết nối
  - name: Tracking_Set_User
    x-displayName: Định danh khách hàng truy cập website
    description: |
      Định danh khách hàng theo email hoặc số điện thoại. Nếu email hoặc số điện thoại chưa tồn tại hệ thống sẽ tự động tạo contact.
      Ví dụ:
      ```javascript
         _goq.push(['setUserId', { email: "customer@customer.com",phone:"84912345678" }]);

        ```
  - name: Tracking_Event
    x-displayName: Theo dõi sự kiện trên website
    description: |
      * Để theo dõi hành vi của khách hàng trên website sử dụng các sự kiện mặc định [**tại đây**](#tag/Default_Event_Types). Nếu cần theo dõi các sự kiện khác (ngoài danh sách trên), vui lòng xem hướng dẫn [**tại đây**](https://docs.gapone.vn/vi/event-tracking)
      * Theo dõi hành vi của khách hàng theo mẫu dưới đây:
      ```javascript
        GaponeScript.trackEvent(event_name,event_data);
      ```
          Thuộc tính | Kiểu dữ liệu | Mô tả
          -----------|-------------|-------
          event_name | string | Tên sự kiện (bắt buộc)
          event_data | object | thuộc tính sự kiện

        Ví dụ:

        ```javascript
            GaponeScript.trackEvent('gapone_web_cart_add',{
              "product_code": "1030971764",
              "sku": "1067924393",
              "quantity": 1,
              "title": "Đầm Sát Nách Phối Bèo",
              "product_name": "Đầm Sát Nách Phối Bèo",
              "price": 12000000,
              "total": 12000000,
              "cart_quantity": 2,
              "total_order": 24000000
            });

          ```
  - name: Tracking_Event_Server
    x-displayName: Theo dõi sự kiện theo server site API (custom event API)
    description: |
      * Sử dụng api này khi không cài đặt mã tracking trên website hoặc theo dõi hành vi của khách hàng với các tham số tùy chọn (custom event).
      * Gọi API theo mẫu dưới đây:

      ```http
      HTTP Request:

          POST /api/event/<siteId> HTTP/1.1
          Host: tracking.gapone.vn
          Authorization: Bearer <token>
          Content-Type: application/json;charset=UTF-8

          {
           "event": "order_create",
           "customer": {
           "email": "hung@gapit.com.vn",
           "phone": "0912345678",
           "cookie_id":"addadadada",
           "FirstName": "hung",
           "LastName": "bui"
           },
           "properties": {
           "order_id": "1234",
           "total_price":1000000,
           "product_id": "1111",
           "product_name":"Quần nam",
           "product_categories": ["Thời Trang", "Quần"],
           "product_brand": "A"
           "order_status":"unconfirmed"
           },
           "time": 1626687244
          }



        HTTP Response:

          HTTP/1.1 202 ACCEPTED
        ```

        * Authorization: Sử dụng xác thực Bearer token, xem chi tiết [**tại đây**](#section/Authentication)

        * Request Params:

          Tham số | Kiểu dữ liệu | Mô tả
          -----------|-------------|-------
          siteId (bắt buộc) | integer | tạo site và lấy id tại đây: https://docs.gapone.vn/vi/site-tracking
        
        * Request Body: 

          Tham số | Kiểu dữ liệu | Mô tả
          -----------|-------------|-------
          event (bắt buộc) | string | Tên sự kiện, tối đa 50 ký tự, không bao gồm dấu cách và các ký tự đặc biệt
          customer (bắt buộc) | object | Thông tin người dùng liên quan đến event, có ít nhất 1 trong 3 thuộc tính: email, phone, cookie_id . Các thuộc tính phải nằm trong danh sách thuộc tính khách hàng tại https://app.gapone.vn/Property/Index?keyWordSearch=&Filter=1
          customer.email  | string | Email của khách hàng
          customer.phone  | string | Số điện thoại của khách hàng. VD: 84912345678
          customer.cookie_id  | string | Cookie id của người dùng nếu có khi gắn tracking trên website. Cookie_id lấy theo cách sau: <br><pre>var cookie_id;<br>_goq.push([ function() { cookie_id = this.getVisitorId(); }]);</pre>
          properties  | object | Thông tin liên quan đến sự kiện, tối đa 20 thuộc tính, hỗ trọ các kiểu dữ liệu:<br> &nbsp;  &bull; string <br> &nbsp;   &bull; integer <br> &nbsp;  &bull; Array of string
          time  | integer (unix timestamp) | Thời gian xảy ra sự kiện. Mặc định sẽ là thời gian gửi request


        * Curl example:
          ```curl

            curl --location -g --request POST 'https://tracking.gapone.vn/api/event/{siteId}' \
            --header 'Authorization: Bearer {BEARER_TOKEN}' \
            --header 'Content-Type: application/json' \
            --data-raw ' {
                         "event": "order_create",
                         "customer": {
                         "email": "hung@gapit.com.vn",
                         "phone": "0912345678",
                         "cookie_id":"addadadada",
                         "FirstName": "hung",
                         "LastName": "bui"
                         },
                         "properties": {
                         "order_id": "1234",
                         "total_price":1000000,
                         "product_id": "1111",
                         "product_name":"Quần nam",
                         "product_categories": ["Thời Trang", "Quần"],
                         "product_brand": "A"
                         "order_status":"unconfirmed" 
                         },
                         "time": 1626687244
                        }'
          ```

x-tagGroups:
  - name: API CALL
    tags:
      - A2P Message
      - Zalo
      - Brandname
      - Contact
      - Product
      - Deal
  - name: Nhận delivery notification từ GAPONE
    tags:
      - dn_overview
      - dn_url
      - dn_request
      - dn_response
      - dn_status
      - dn_example
  - name: Website Tracking Code API
    tags:
      - Tracking_Introduction
      - Tracking_Install
      - Gapone_Object
      - Tracking_Event
      - Default_Event_Types
  - name: Tracking Code API
    tags:
      - Tracking_Event_Server
  - name: Channel Messages
    tags:
      - SMS_Channel
      - Viber_Channel
      - Email_Channel
      - Zalo_Channel
      - Zalo_RSA_Channel
      - Zalo_Journey_Channel
      - Zalo_OA_Channel
      - Whatsapp_Channel
  - name: Danh sách mã trạng thái
    tags:
      - error_status
      - delivery_status
      - order_status
  - name: Câu hỏi thường gặp
    tags:
      - FAQs


```
