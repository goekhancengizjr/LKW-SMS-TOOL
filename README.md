<!DOCTYPE html>
<html lang="de">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>LKW-SMS-Tool mit PDF & SMS</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <style>
        body { font-family: Arial, sans-serif; text-align: center; margin: 20px; }
        input, textarea, select, button { width: 90%; margin: 5px 0; padding: 10px; font-size: 16px; }
        button { background-color: #28a745; color: white; border: none; cursor: pointer; }
        button:hover { background-color: #218838; }
        table { width: 100%; border-collapse: collapse; margin-top: 20px; }
        th, td { border: 1px solid #ddd; padding: 10px; text-align: left; }
        th { background-color: #f4f4f4; }
        tr:hover { background-color: #f8f8f8; }
        .confirmation { color: green; font-weight: bold; margin-top: 10px; display: none; }
    </style>
</head>
<body>

    <h2>LKW-SMS-Tool</h2>
    <form id="smsForm">
        <input type="time" id="stempelzeit" placeholder="Stempelzeit (hh:mm)" required>
        <input type="text" id="kennzeichen" placeholder="Kennzeichen" required>
        <input type="text" id="ecargo" placeholder="eCargo-Nummer" required>
        
        <label for="lkwTyp">LKW-Typ:</label>
        <select id="lkwTyp">
            <option value="Standard">Standard</option>
            <option value="Mega">Mega</option>
        </select>

        <label for="gefahrgut">Gefahrgut:</label>
        <select id="gefahrgut">
            <option value="Nein">Nein</option>
            <option value="Ja">Ja</option>
        </select>

        <input type="text" id="spediteur" placeholder="Spediteur">
        <input type="tel" id="phoneNumber" placeholder="Telefonnummer" required>
        <textarea id="bemerkung" placeholder="Bemerkungen" rows="2"></textarea>
        <textarea id="message" placeholder="SMS-Nachricht" rows="4" required></textarea>
        
        <button type="submit">Speichern & PDF erstellen</button>
    </form>

    <h3>Auftrags-Report</h3>
    <table>
        <thead>
            <tr>
                <th>Stempelzeit</th>
                <th>Kennzeichen</th>
                <th>eCargo-Nummer</th>
                <th>LKW-Typ</th>
                <th>Gefahrgut</th>
                <th>Spediteur</th>
                <th>Bemerkung</th>
                <th>SMS-Nachricht</th>
                <th>Telefonnummer</th>
                <th>PDF</th>
                <th>SMS senden</th>
            </tr>
        </thead>
        <tbody id="orderTable"></tbody>
    </table>

    <p id="smsConfirmation" class="confirmation"></p>

    <script>
        const { jsPDF } = window.jspdf;
        const orderData = [];

        document.getElementById("smsForm").addEventListener("submit", function(event) {
            event.preventDefault();

            let order = {
                stempelzeit: document.getElementById("stempelzeit").value,
                kennzeichen: document.getElementById("kennzeichen").value,
                ecargo: document.getElementById("ecargo").value,
                lkwTyp: document.getElementById("lkwTyp").value,
                gefahrgut: document.getElementById("gefahrgut").value,
                spediteur: document.getElementById("spediteur").value,
                phoneNumber: document.getElementById("phoneNumber").value,
                bemerkung: document.getElementById("bemerkung").value,
                message: document.getElementById("message").value
            };

            orderData.push(order);
            displayOrderTable();
        });

        function displayOrderTable() {
            const tableBody = document.getElementById("orderTable");
            tableBody.innerHTML = "";

            orderData.forEach((order, index) => {
                const row = document.createElement("tr");
                row.innerHTML = `
                    <td>${order.stempelzeit}</td>
                    <td>${order.kennzeichen}</td>
                    <td>${order.ecargo}</td>
                    <td>${order.lkwTyp}</td>
                    <td>${order.gefahrgut}</td>
                    <td>${order.spediteur}</td>
                    <td>${order.bemerkung}</td>
                    <td>${order.message}</td>
                    <td>${order.phoneNumber}</td>
                    <td><button onclick="generatePDF(${index})">📄 PDF</button></td>
                    <td><button onclick="sendSMS(${index})">📩 SMS</button></td>
                `;
                tableBody.appendChild(row);
            });
        }

        function generatePDF(index) {
            const order = orderData[index];
            const doc = new jsPDF();

            doc.setFontSize(16);
            doc.text("Auftragsbericht", 10, 10);
            doc.setFontSize(12);

            let text = `
Stempelzeit: ${order.stempelzeit}
Kennzeichen: ${order.kennzeichen}
eCargo-Nummer: ${order.ecargo}
LKW-Typ: ${order.lkwTyp}
Gefahrgut: ${order.gefahrgut}
Spediteur: ${order.spediteur}
Telefonnummer: ${order.phoneNumber}
Bemerkung: ${order.bemerkung}
SMS-Nachricht: ${order.message}
            `;

            doc.text(text, 10, 30);
            
            const pdfBlob = doc.output('blob');
            const downloadLink = document.createElement('a');
            downloadLink.href = URL.createObjectURL(pdfBlob);
            downloadLink.download = `Auftrag_${order.ecargo}.pdf`;

            document.body.appendChild(downloadLink);
            downloadLink.click();
            document.body.removeChild(downloadLink);
        }

        function sendSMS(index) {
            const order = orderData[index];
            const smsBody = encodeURIComponent(`LKW-Auftrag:\nStempelzeit: ${order.stempelzeit}\nKennzeichen: ${order.kennzeichen}\nAuftragsnr.: ${order.ecargo}\nLKW-Typ: ${order.lkwTyp}\nGefahrgut: ${order.gefahrgut}\nSpediteur: ${order.spediteur}\nBemerkung: ${order.bemerkung}`);
            const smsLink = `sms:${order.phoneNumber}?body=${smsBody}`;

            // SMS-Link öffnen
            window.location.href = smsLink;

            // SMS-Bestätigung anzeigen
            const confirmation = document.getElementById("smsConfirmation");
            confirmation.textContent = `✅ SMS erfolgreich gesendet an ${order.phoneNumber}`;
            confirmation.style.display = "block";

            // Bestätigung nach 3 Sekunden ausblenden
            setTimeout(() => {
                confirmation.style.display = "none";
            }, 3000);
        }
    </script>

</body>
</html>
