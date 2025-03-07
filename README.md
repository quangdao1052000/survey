<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Khảo sát lòng tin trong công việc</title>
  <link href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css" rel="stylesheet">
  <style>
    body { font-family: Arial, sans-serif; background-color: #f8f9fa; }
    .survey-container {
      max-width: 800px;
      margin: 50px auto;
      background-color: white;
      padding: 30px;
      border-radius: 10px;
      box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.1);
    }
    .progress-bar { height: 20px; }
    .question { font-size: 18px; margin-bottom: 20px; }
    .answers { margin-bottom: 20px; }
    .btn-custom { background-color: #007bff; color: white; border: none; }
    .btn-custom:disabled { background-color: gray; }
  </style>
</head>
<body>
  <div class="survey-container">
    <h2 class="text-center">Khảo sát về lòng tin trong công việc</h2>
    <div class="progress">
      <div id="progress" class="progress-bar progress-bar-striped" role="progressbar" style="width: 0%;" aria-valuenow="0" aria-valuemin="0" aria-valuemax="100">0%</div>
    </div>
    <div id="question-container"></div>
    <div class="text-center">
      <button id="next-btn" class="btn btn-custom" onclick="nextQuestion()" disabled>Tiếp theo</button>
    </div>
  </div>

  <!-- Các thư viện cần thiết -->
  <script src="https://code.jquery.com/jquery-3.5.1.slim.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/@popperjs/core@2.9.3/dist/umd/popper.min.js"></script>
  <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/js/bootstrap.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.17.0/xlsx.full.min.js"></script>
  
  <script>
    const questions = [
      "1. Tôi cảm thấy tin tưởng vào đồng nghiệp của mình.",
      "2. Tôi tin tưởng vào khả năng lãnh đạo của quản lý.",
      "3. Tôi cảm thấy các quyết định trong công ty đều công bằng.",
      "4. Tôi cảm thấy an toàn khi chia sẻ ý kiến trong công ty.",
      "5. Tôi tin tưởng rằng các cam kết công ty đưa ra sẽ được thực hiện.",
      "6. Tôi cảm thấy công ty coi trọng ý kiến đóng góp của nhân viên.",
      "7. Tôi tin rằng công ty luôn hành động vì lợi ích của nhân viên.",
      "8. Tôi tin rằng các quyết định trong công ty được đưa ra một cách minh bạch.",
      "9. Tôi cảm thấy tin tưởng vào khả năng giải quyết vấn đề của đồng nghiệp.",
      "10. Tôi cảm thấy môi trường làm việc tại công ty là trung thực."
    ];
    
    let currentQuestion = 0;
    const responses = [];
    
    function showQuestion() {
      console.log("showQuestion: currentQuestion =", currentQuestion);
      if (currentQuestion < questions.length) {
        const question = questions[currentQuestion];
        document.getElementById('question-container').innerHTML = `
          <div class="question">${question}</div>
          <div class="answers">
            <div class="form-check">
              <input type="radio" class="form-check-input" name="answer" value="1" onclick="enableNextButton()">
              <label class="form-check-label">Rất không đồng ý</label>
            </div>
            <div class="form-check">
              <input type="radio" class="form-check-input" name="answer" value="2" onclick="enableNextButton()">
              <label class="form-check-label">Không đồng ý</label>
            </div>
            <div class="form-check">
              <input type="radio" class="form-check-input" name="answer" value="3" onclick="enableNextButton()">
              <label class="form-check-label">Trung lập</label>
            </div>
            <div class="form-check">
              <input type="radio" class="form-check-input" name="answer" value="4" onclick="enableNextButton()">
              <label class="form-check-label">Đồng ý</label>
            </div>
            <div class="form-check">
              <input type="radio" class="form-check-input" name="answer" value="5" onclick="enableNextButton()">
              <label class="form-check-label">Rất đồng ý</label>
            </div>
          </div>
        `;
        // Reset trạng thái nút "Tiếp theo" khi hiển thị câu hỏi mới
        document.getElementById('next-btn').disabled = true;
        updateProgress();
      }
    }
    
    function enableNextButton() {
      console.log("enableNextButton called");
      document.getElementById('next-btn').disabled = false;
    }
    
    function nextQuestion() {
      console.log("nextQuestion called");
      const selectedAnswer = document.querySelector('input[name="answer"]:checked');
      if (!selectedAnswer) {
        alert('Vui lòng chọn câu trả lời');
        return;
      }
      console.log("Selected answer:", selectedAnswer.value);
      responses.push(selectedAnswer.value);
      currentQuestion++;
      
      if (currentQuestion < questions.length) {
        showQuestion();
      } else {
        // Khi trả lời hết các câu hỏi, chuyển nút thành "Hoàn thành"
        document.getElementById('next-btn').innerText = 'Hoàn thành';
        // Đảm bảo nút được bật để có thể nhấn
        document.getElementById('next-btn').disabled = false;
        // Gán sự kiện xuất file Excel khi nhấn nút "Hoàn thành"
        document.getElementById('next-btn').onclick = function() {
          exportToExcel();
        };
      }
    }
    
    function updateProgress() {
      const progress = (currentQuestion / questions.length) * 100;
      document.getElementById('progress').style.width = progress + '%';
      document.getElementById('progress').innerText = Math.round(progress) + '%';
    }
    
function exportToExcel() {
  console.log("Exporting to Excel");
  // Xây dựng dữ liệu dưới dạng mảng các mảng
  let data = [["Câu hỏi", "Đáp án"]];
  questions.forEach((question, index) => {
    data.push([question, responses[index] || ""]);
  });
  
  // Tạo workbook và chuyển đổi mảng dữ liệu thành worksheet
  const wb = XLSX.utils.book_new();
  const ws = XLSX.utils.aoa_to_sheet(data);
  XLSX.utils.book_append_sheet(wb, ws, "Khảo sát");
  XLSX.writeFile(wb, "KhaoSatLongTin.xlsx");
}


    
    // Hiển thị câu hỏi đầu tiên
    showQuestion();
  </script>
</body>
</html>
