# vaccine_site
## 홈화면
![image](https://github.com/hwan06/vaccine_site/assets/114748934/4d0b782e-cc1d-47f8-b9b7-e3e02cee1a9c)

## 백신예약 화면
![image](https://github.com/hwan06/vaccine_site/assets/114748934/8ade95ec-42de-4b2b-a220-3de752123f78)   
### 예약번호 자동 발생을 위한 예약번호 max값 + 1을 저장
```java
<%
	Connection conn = DBconnect.getConnection();
	String sql = "select max(resvno) from tbl_vaccresv_202108";
	PreparedStatement ps = conn.prepareStatement(sql);
	ResultSet rs = ps.executeQuery();
	rs.next();
	
	int num = rs.getInt(1) + 1;
%>
```
### 표현식을 이용하여 num값을 삽입
```java
<th>예약번호</th>
<td><input type="text" value="<%=num %>" name="	" readonly="readonly">
```
### 유효성 검사를 위한 js코드
```js
function check() {
		if(!document.data.jumin.value) {
			alert("주민번호를 입력해주세요.");
			data.jumin.focus();
			return false;
		}else if(!document.data.v_code.value) {
			alert("백신을 선택해주세요.");
			data.v_code.focus();
			return false;
		}else if(!document.data.hospital.value) {
			alert("병원을 선택해주세요.");	
			data.hospital.focus();
			return false;
		}else if(!document.data.j_date.value) {
			alert("예약 날짜를 입력해주세요.");
			data.j_date.focus();
			return false;
		}else if(!document.data.j_time.value) {
			alert("예약 시간을 입력해주세요.");
			data.j_time.focus();
			return false;
		}
		return true;
	}
```
## 백신예약조회 화면
![image](https://github.com/hwan06/vaccine_site/assets/114748934/7f77656e-06ce-466f-8d15-e9d1d41ba909)
### 위의 사진에 입력한 예약번호가 있는 경우에 형식에 맞추어 데이터를 검색하는 SQL코드 
```java
<%
	int in_resvno = Integer.parseInt(request.getParameter("j_num"));
	Connection conn = DBconnect.getConnection();
	String sql = "select tv.resvno, tj.jumin, "  
	 +" case" 
	 +" when substr(tj.jumin, 8, 1) = '1' then '남' "
	 +" else '여' end as gender, "
	 +" tv.hospcode, tv.resvdate, " 
	 +" substr(to_char(tv.resvdate, 'yyyymmdd'), 1, 4) || '년' "
	 +" || substr(to_char(tv.resvdate, 'yyyymmdd'), 5, 2) || '월' "
	 +" || substr(to_char(tv.resvdate, 'yyyymmdd'), 7, 2) || '일' as time1, "
	 +" case "
	 +" when tv.vcode = 'V001' then 'A백신' "
	 +" when tv.vcode = 'V002' then 'B백신' "
	 +" else 'C백신' "
	 +" end as vcode, "
	 +" case "
	 +" when hospaddr = '10' then '서울' "
	 +" when hospaddr = '20' then '대전' "
	 +" when hospaddr = '30' then '대구' "
	 +" else '광주' "
	 +" end as hospaddr "
	 +" from tbl_jumin_202108 tj, tbl_vaccresv_202108 tv, tbl_hosp_202108 th "
	 +" where tj.jumin = tv.jumin and tv.hospcode = th.hospcode and tv.resvno = ?";
	 PreparedStatement ps = conn.prepareStatement(sql);
	 ps.setInt(1, in_resvno);
	 ResultSet rs = ps.executeQuery();
%>	
```
### 만약 rs에 값이 있다면, 테이블에 가져온 데이터를 출력. 아니라면, 조회결과없음 메시지 표시
```java
<%if(rs.next()) { %>
	<table class = "table">
		<tr>
			<th>예약번호</th>
			<th>성명</th>
			<th>성별</th>
			<th>병원이름</th>
			<th>예약날짜</th>
			<th>예약시간</th>
			<th>백신코드</th>
			<th>병원코드</th>
		</tr>
		
		<tr>
			<td><%=Integer.parseInt(rs.getString("resvno")) %></td>
			<td><%=rs.getString("jumin") %></td>
			<td><%=rs.getString("gender") %></td>
			<td><%=rs.getString("hospcode") %></td>
			<td><%=rs.getString("resvdate") %></td>
			<td><%=rs.getString("time1") %></td>
			<td><%=rs.getString("vcode") %></td>
			<td><%=rs.getString("hospaddr") %></td>
		</tr>
	</table>
<%} else {%>
	<h2 style="text-align: center;">예약번호 <%=in_resvno %>로 조회된 결과가 없습니다.</h2>
<%} %>
```
## 백신예약현황 화면
![image](https://github.com/hwan06/vaccine_site/assets/114748934/b07c9f36-5a2e-4b40-9dff-2f17f9694866)   
### 외부조인과 nvl을 이용하여 데이터값이 0인 데이터까지 출력하는 SQL코드
``` java
<%
	Connection conn = DBconnect.getConnection();
	String sql = "select th.hospaddr, "
	+" case "
	+" when th.hospaddr = '10' then '서울' "
	+" when th.hospaddr = '20' then '대전' "
	+" when th.hospaddr = '30' then '대구' "
	+" else '광주' "
	+" end as hospaddr1, "
	+" nvl(count(tv.hospcode),0) as count1 "
	+" from tbl_hosp_202108 th, tbl_vaccresv_202108 tv "
	+" where th.hospcode = tv.hospcode(+) "
	+" group by th.hospaddr "
	+" order by count1 desc"; 
	PreparedStatement ps = conn.prepareStatement(sql);
	ResultSet rs = ps.executeQuery();
	
	int sum = 0;
%>
```
### count1을 자바를 이용하여 누적합계를 구해 총합계를 구하여 테이블에 출력.
```java
<%while(rs.next()) { sum+=Integer.parseInt(rs.getString("count1"));%>
		<tr>
			<td><%=rs.getString("hospaddr") %></td>
			<td><%=rs.getString("hospaddr1") %></td>
			<td><%=rs.getString("count1") %></td>			
		</tr>
		<%} %>
```


