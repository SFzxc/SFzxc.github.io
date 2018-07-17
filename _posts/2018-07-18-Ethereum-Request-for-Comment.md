---
layout: post
title:  "Ethereum Request forComment"
date:   2018-07-18 01:00:00 +0700
categories: [Solidity, SmartContract, ERC]
comment: true
---

ERC20 là một quy chuẩn chung mà một Token Contract trong Ethereum nên tuân theo, nó được đề nghị bởi 2 developer Fabian Vogelsteller và Vitalik Buterin vào năm 2015.

ERC20 đặt ra các Methods và các Events cơ bản mà một Token Contract nên có như:

```
    function name() view returns (string name)
    function symbol() view returns (string symbol)
    function decimals() view returns (uint8 decimals)
    function totalSupply() view returns (uint256 totalSupply)
    function balanceOf(address _owner) view returns (uint256 balance)
    function transfer(address _to, uint256 _value) returns (bool success)
    function transferFrom(address _from, address _to, uint256 _value) returns (bool success)
    function approve(address _spender, uint256 _value) returns (bool success)
    function allowance(address _owner, address _spender) view returns (uint256 remaining)

    event Transfer(address indexed _from, address indexed _to, uint256 _value)
    event Approval(address indexed _owner, address indexed _spender, uint256 _value)
```

Hầu hết các tokens được tạo để  ICO trên Ethereum đều dựa tuân theo chuẩn ERC20. Lợi ích của việc tuân theo một quy chuẩn cũng giống như ta viết apis theo chuẩn RESTful vậy. Việc lập trình và tương tác giữa các ứng dụng hỗ trợ xoay quanh cũng trở nên đơn giản hơn. Giả dụ như việc Ethereum Block Explorer muốn theo dõi sự kiện event Transfer để hiển thị mỗi holder đang có token balance là bao nhiêu. Nếu mỗi token contract đặt tên event theo mỗi cách khác nhau thì rất khó để tracking.

**Token Contract là gì?**

Về mặt bản chất, token contract chính là một smart contract, mà trong đó chứa những mapping giữa account addresses và 1 giá trị balance của address đó. Balance này có thể là một đại diện cho một giá trị gì đó ở ngoài đời thật, chẳng hạn như tiền tệ, cấp bậc, độ cống hiến chẳng hạn. Và đơn vị của balance này người ta gọi là Token.

Khi token được chuyển từ một account này sang một account khác thì Token Contract sẽ update accounts balance đó tương tự như tài khoản ngân hàng vậy.

Thường khi tạo một token contract, người ta sẽ khởi tạo tổng số lượng token ban đầu là một số cố định nào đó. Tất nhiên cũng có một số contract có phép tăng hoặc giảm tổng số lượng này tuỳ thuộc vào business logic như việc minting token mới hay burining existing tokens. Ngoài ra còn có một cách burning tokens khá thú vị đó là gửi tokens vào một địa chỉ mà ở đó chúng ta không biết private key là gì. Có một địa chỉ khá nổi tiếng là the 0 address. Với việc không biết pk thì đồng nghĩa với việc chúng ta hoặc ai khác có thể sở hữu và sử dụng được nó.

**ERC-20 token contract**

Trong phần này chúng ta sẽ đi sâu một chút để hiểu rõ hơn ý nghĩa của các functions và events theo chuẩn ERC-20.

- name thì như tên gọi của nó, hiện thị tên đầy đủ của một token contract.
- symbol chính tên rút gọn, mã token gồm từ 3 đến 4 chữ cái mà ta thường thấy trên các sàn giao dịch tokens.
- decimals Chính là thông số cho chúng ta biết được token cần này có thể chia nhỏ hơn thành bao nhiêu phần. Thường sẽ chạy từ 0 đến 18 (bằng đúng với cách chia của ETH) hoặc có thể nhỏ hơn. Tuỳ thuộc vào business logic để quyết định xem nên xét nó là giá trị bao nhiêu. Chẳng hạn token đại diện cho một thực thể không thể tách rời như con người chẳng hạn thì nên set decimals là 0.
- totalsupply như đã nói ở trên chính là tổng số token balances mà chúng ta thường khởi tạo khi tạo token contract.
- balanceOf()  trả về giá trị số lượng token mà address đó đang nắm giữ, bất cứ người nào cũng có thể tra cứu vì dữ liệu này là public.
- transfer() chính là function giúp cho việc chuyển token từ một address này sang một address khác.
- Có một cách khác để transfer token nữa đó là dùng Approve and TransferFrom. Tại sao lại nãy sinh thêm 2 hàm này. Cái này khá là thú vị. Chúng ta giả dụ một trường hợp có liên quan như là Decentralized Exchange chẳng hạn. Ví như bạn muốn đổi 200 EXAMPLE token để nhận 10 DEX token chẳng hạn.  DEX contract có một hàm gọi là deposit để nhận tiền. Nếu chúng ta cố tình send 200 EXAMPLE token vào DEX contract bằng cách gọi hàm EXAMPLE.transfer. Chắc chắn chúng ta sẽ mất oan 200 tokens này. Vì DEX contract không biết người gửi là ai vì nó không có mapping address<->balance cho người gửi.
  Thay vào đó, có một giải pháp đó là hàm DEX deposit sẽ chuyển tokens từ tài khoản của bạn vào tài khoản của họ. Họ sẽ thực thi EXAMPLE.transferFrom(yourAccount,DEXccount,200) bên trong hàm deposit của họ.Nhưng tất nhiên họ phải được sự cho phép của bạn bằng cách bạn uỷ quyền cho EXAMPLE contract rằng DEX có thể được một số tokens nhất định từ account của bạn bằng hàm approve.

**ERC-223**

 	ERC-20 cung cấp những quy chuẩn tốt để xây dựng token contract nhưng nó cũng tồn tại một số điểm lỗi khá nghiệm trọng như việc tokens có thể mất nếu địa chỉ nhận là một contract address chứ không phải là account address. ERC-223 chính là giải pháp cứu cánh cho vấn đề đó bằng cách thêm vào đó một số điều kiện nhằm phân  biệt và kiểm tra địa chỉ nhận có phải contract address không ?!. Nếu có sẽ không cho phép thực hiện. Và thêm một số thay đổi quan trọng khác mà mọi người có thể đọc thêm tại  đây .

**Tài liệu tham khảo**

- https://github.com/ethereum/EIPs/blob/master/EIPS/eip-20.md
- https://github.com/Dexaran/ERC223-token-standard
