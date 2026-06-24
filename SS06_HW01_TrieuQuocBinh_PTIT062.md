# BÀI 1: Thực hành Tối ưu hóa Lưu trữ Giỏ hàng (What-if Scenario & Multiple Options)

### 1. Phân tích bối cảnh và lý do cần thiết kế prompt đa chiều
Trong kiến trúc phần mềm, không có giải pháp nào là hoàn hảo tuyệt đối (Silver Bullet). Việc lưu trữ giỏ hàng (Shopping Cart) đối mặt với bài toán kinh điển: Cân bằng giữa tốc độ (Hiệu năng) và độ tin cậy (Tính bền vững của dữ liệu).
Nếu chỉ dùng một prompt đơn giản như *"Hãy viết code lưu giỏ hàng bằng Redis"*, AI sẽ chỉ tạo ra một đoạn code thô mà bỏ qua việc đánh giá rủi ro hệ thống. Bằng cách thiết kế một prompt đa chiều (áp dụng Multiple Options, Trade-offs và What-if Scenario), chúng ta ép AI phải suy luận như một Kiến trúc sư thực thụ: phải phân tích ưu/nhược điểm để thuyết phục tại sao lại chọn công nghệ đó, và quan trọng nhất là phải có kịch bản dự phòng (Disaster Recovery) khi công nghệ đó "sập". Điều này giúp giải pháp đưa ra mang tính thực tiễn và an toàn cao khi đưa lên môi trường Production.

---

### 2. Nội dung Prompt tối ưu thiết kế (Chứa 5 thành phần cốt lõi)

"**[Role]** Hãy đóng vai trò là một Kiến trúc sư Hệ thống (System Architect) giàu kinh nghiệm của sàn thương mại điện tử SpeedyCart.
**[Context]** Hệ thống đang gặp nút thắt cổ chai (bottleneck): Giỏ hàng tạm thời có tần suất đọc/ghi cực kỳ lớn khi người dùng thêm/bớt sản phẩm, khiến SQL Database truyền thống bị nghẽn và làm tăng độ trễ trang web. Tôi cần một giải pháp lưu trữ tối ưu hơn.
**[Goal]** Nhiệm vụ của bạn là tư vấn kiến trúc lưu trữ giỏ hàng thông qua 3 bước:
1. **(Multiple Options & Trade-offs):** Đề xuất 3 phương án công nghệ (SQL Database, In-memory Redis Cache, Client-side Cookie/Session). Lập bảng so sánh chi tiết theo các tiêu chí: Tốc độ đọc/ghi, Tính đồng nhất dữ liệu, Khả năng chịu lỗi/mất mát dữ liệu, và Độ phức tạp triển khai.
2. **(What-if Scenario):** Phân tích kịch bản: *'Chuyện gì xảy ra nếu chọn Redis nhưng cụm máy chủ Redis đột ngột bị crash hoặc mất nguồn điện?'* Hãy tư vấn giải pháp để đảm bảo giỏ hàng không bị biến mất hoàn toàn bằng cách kết hợp cơ chế Persistence (RDB/AOF) của Redis và cơ chế fallback đồng bộ tạm xuống SQL.
3. **(Implementation):** Cung cấp mã nguồn Java Spring Boot cấu hình `RedisTemplate` an toàn, chuyên nghiệp.
   **[Constraint]** Mã nguồn phải thiết lập đúng các cấu hình Serializer (như `StringRedisSerializer` và `GenericJackson2JsonRedisSerializer`) để dữ liệu giỏ hàng có thể đọc được trên hệ thống quản trị, thay vì bị mã hóa thành mảng byte khó hiểu.
   **[Format]** Trình bày theo 3 phần rõ ràng: Bảng so sánh (Markdown), Phân tích kịch bản giả định, và Mã nguồn Java."

---

### 3. Minh chứng chạy thực tế (Text log phản hồi của AI)

```text
Chào bạn, để giải quyết bài toán thắt cổ chai (bottleneck) của SpeedyCart, chúng ta cần chuyển đổi mô hình lưu trữ giỏ hàng từ việc "ghi trực tiếp xuống đĩa" (Disk-based) sang "ghi trên bộ nhớ tạm" (In-memory/Client-side). Dưới đây là tư vấn kiến trúc chi tiết:

#### Phần 1: Bảng so sánh các phương án lưu trữ giỏ hàng (Trade-offs)

| Tiêu chí | 1. SQL Database (Hiện tại) | 2. Client-side (Cookie / LocalStorage) | 3. Redis (In-memory Cache) |
| :--- | :--- | :--- | :--- |
| **Bản chất lưu trữ** | Ổ cứng máy chủ (Disk) | Trình duyệt của người dùng | Bộ nhớ RAM máy chủ |
| **Tốc độ đọc/ghi** | Rất chậm (Gây nghẽn hệ thống) | Cực nhanh (Không tốn băng thông server) | Rất nhanh (Độ trễ mili-giây) |
| **Tính đồng nhất dữ liệu**| Tuyệt đối (Đồng bộ mọi thiết bị) | Kém (Đăng nhập máy khác sẽ bị mất giỏ hàng) | Tuyệt đối (Server quản lý tập trung) |
| **Khả năng chịu lỗi** | Rất cao (Có transaction, backup) | Kém (Người dùng xóa cookie là mất sạch) | Trung bình - Cao (RAM mất điện sẽ xóa sạch nếu không cấu hình) |
| **Độ phức tạp triển khai**| Thấp | Thấp (Nhưng bị giới hạn dung lượng ~4MB) | Cao (Cần bảo trì thêm cụm máy chủ Cache) |

**Kết luận:** Phương án **Redis (In-memory Cache)** là sự lựa chọn cân bằng hoàn hảo nhất: Đảm bảo tốc độ cực cao, chịu tải tốt mà vẫn giữ được tính đồng nhất dữ liệu khi khách hàng đổi thiết bị.

#### Phần 2: Phân tích kịch bản rủi ro (What-if Scenario)

**Kịch bản:** *Cụm máy chủ Redis bị crash hoặc mất điện đột ngột.*
Vì Redis lưu dữ liệu trực tiếp trên RAM, nếu mất điện, toàn bộ giỏ hàng của hàng triệu khách hàng đang online sẽ bốc hơi ngay lập tức. Để giải quyết, tôi đề xuất kiến trúc đảm bảo an toàn sau:
1. **Sử dụng cơ chế Persistence của Redis (RDB + AOF):**
   - Kích hoạt RDB (Redis Database Backup) để chụp ảnh (snapshot) dữ liệu giỏ hàng định kỳ mỗi 5 phút.
   - Kết hợp AOF (Append Only File) để ghi lại từng câu lệnh thao tác giỏ hàng xuống đĩa. Nếu mất điện, khi server khởi động lại, Redis sẽ tự động đọc file AOF/RDB để phục hồi 99% dữ liệu giỏ hàng lên RAM.
2. **Cơ chế Fallback ghi đệm xuống SQL (Asynchronous Sync):**
   - Không ghi SQL trực tiếp mỗi khi khách bấm "Thêm vào giỏ", nhưng ta sẽ thiết lập một luồng ngầm (Background Worker/Message Queue). 
   - Khi giỏ hàng có sự thay đổi lớn hoặc khách hàng chuyển sang bước "Thanh toán" (Checkout), hệ thống sẽ đồng bộ bất đồng bộ (async sync) dữ liệu từ Redis xuống SQL. 
   - Nếu API gọi Redis bị lỗi (Timeout/Connection Refused), hệ thống sẽ tự động bẫy lỗi (try-catch) và kích hoạt chế độ Fallback: Chuyển hướng đọc/ghi giỏ hàng tạm thời thẳng vào SQL Database cho đến khi Redis được sửa xong (Fail-open).

#### Phần 3: Mã nguồn cấu hình RedisTemplate an toàn trong Spring Boot

Dưới đây là cấu hình Spring Boot để kết nối và chuẩn hóa dữ liệu giỏ hàng (Shopping Cart) dưới dạng JSON thay vì lưu chuỗi byte mặc định của Java, giúp dễ dàng debug trên Redis UI.

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

@Configuration
public class RedisConfig {

    @Bean
    public RedisTemplate<String, Object> cartRedisTemplate(RedisConnectionFactory connectionFactory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(connectionFactory);

        // 1. Cấu hình Serializer cho Key (Sử dụng String để key dễ đọc, vd: "cart:user_id:123")
        StringRedisSerializer stringSerializer = new StringRedisSerializer();
        template.setKeySerializer(stringSerializer);
        template.setHashKeySerializer(stringSerializer);

        // 2. Cấu hình Serializer cho Value (Chuyển đổi Object CartItem thành chuỗi JSON tự động)
        // Điều này rất quan trọng để tránh lỗi lưu Byte Array khó đọc trong môi trường Production
        GenericJackson2JsonRedisSerializer jsonSerializer = new GenericJackson2JsonRedisSerializer();
        template.setValueSerializer(jsonSerializer);
        template.setHashValueSerializer(jsonSerializer);

        template.afterPropertiesSet();
        return template;
    }
}