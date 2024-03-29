# baskovolodia.github.io<!DOCTYPE html>
<html>
<head lang="ru">
    <meta charset="UTF-8">
    <meta name="author" content="Ищенко Иванна">
    <meta name="description" content="Этот калькулятор позволяет  производить приблизительный расчет платежей по кредиту при ипотеке онлайн, и сравнивать, какие платежи выгоднее">
    <meta name="keywords" content="кредитный калькулятор ипотека, онлайн, калькуляторы банков">
    <title>Кредитный калькулятор</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.3.1/jquery.min.js"
        integrity="sha256-FgpCb/KJQlLNfOu91ta32o/NMZxltwRo8QtmkMRdAu8="
        crossorigin="anonymous"></script>
    <style>
        body {
            text-align: center;
            font-family: "Segoe UI", Ubuntu, "Droid Sans", sans-serif;
        }
        form, table {
            text-align: left;
            margin: auto;
        }
        form {
            padding: 2mm;
            border-radius: 2mm;
            width: 12cm;
            background-color: rgba(0, 100, 205, 0.8);
        }
        button {
            background-color: rgba(0, 104, 83, 0.1);
        }
        button, input, select {
            border-radius: 1mm;
        }
        form, button, input, select {
            border: 1px solid #BC8F8F;
        }
        form {
            border-width: 2px;
        }
        form > div {
            display: table-row;
        }
        form > div > div {
            display: table-cell;
            vertical-align: middle;
            padding: 1mm 2mm;
        }
        #credit span {
            color: rgb(160,160,160);
        }
        input {
            width: 16mm;
            padding-left: 1mm;
        }
        button, select {
            width: 24mm;
        }
        button {
            margin: 2mm 1mm 1mm 1mm;
        }
        #buttons {
            display: block;
            text-align: right;
        }
        table {
            border-collapse: collapse;
            text-align: right;
            margin-top: 6mm;
        }
        table td, table th {
            border: 2px solid black;
            padding: 2mm 3mm;
        }
        table span {
            color: rgb(148,59,0);
        }
    </style>
</head>
<body>
<h3>Кредитный калькулятор: введите данные для расчет</h3>
<form id="credit">
    <div>
        <div>Размер кредита</div>
        <div><input id="amount" type="number" min="100" max="500000" pattern="[0-9]{2,5}"
            required title="Введите сумму кредита"/></div>
        <div>грн</div>
    </div>
    <div>
        <div>Процентная ставка по кредиту</div>
        <div><input id="interest" type="text" pattern="[0-9]{1,2}([\.\,][0-9]{1,2})?"
            required title="Введите процентную ставку по кредиту"/></div>
        <div>
            <select id="interest_term" title="Введите процентную ставку по кредиту">
                <option value="month">% в мес.</option>
                <option value="year">% в год</option>
            </select>
        </div>
    </div>
    <div>
        <div>Срок кредита</div>
        <div><input id="maturity" type="number" min="1" max="48" pattern="[0-9]{1,2}" required title="Введите срок кредита"/></div>
        <div>
            <select id="maturity_unit" title="Выберите периодичность начисления процентов">
                <option value="month">мес.</option>
                <option value="year">лет</option>
            </select>
        </div>
    </div>
    <div id="buttons">
            <button type="submit" id="calculate">Рассчитать</button>
            <button type="button" id="reset">Сбросить</button>
    </div>
</form>
<table cellspacing="0">
    <thead>
    <tr>
        <th>Мес.</th>
        <th>Всего</th>
        <th>% по кредиту</th>
        <th>В погашение долга</th>
        <th>В погашение процентов</th>
        <th>Остаток</th>
    </tr>
    </thead>
    <tbody id="months"></tbody>
</table>


<script>
    function calculate(amount, interest, maturity) {
        var table = $('#months');
        table.empty();
        var vat_amount = amount * 1.2; // стоимость оборудования * 1.2 = сумма с ПДВ
        var bank_service = vat_amount * 0.016; // сумму с ПДВ *1.6= % банка за услуги
        var credit = vat_amount + bank_service; // сумма с ПДВ +%банка = тело кредита (кредит)
        var monthly_paid = credit / maturity; // кредит / срок погашения = сумму к уплате ежемесячно
        for (var month = 1; month <= maturity; month++) {
            var row = $('<tr></tr>');
            function cell(value, round) {
                if (isNaN(value) || !isFinite(value))
                    value = 'ошибка';
                else if (round)
                    value = Math.ceil(value);
                else {
                    var c = Math.ceil(value * 100) / 100;
                    var r = Math.floor(c);
                    var coins = Math.ceil((c - r) * 100);
                    coins = ',' + (coins + '00').slice(0, 2);
                    coins = '<span>' + coins + '</span>';
                    value = Math.floor(value) + coins;
                }
                $('<td></td>').html(value)
                        .appendTo(row);
            }
            cell(month, true);
            cell(credit);
            var loan = credit * interest / 100; // кредит * 1,9% = % по кредиту
            cell(loan);
            cell(monthly_paid);
            var monthly_fee = monthly_paid + loan; // сумма к уплате ежемесечно+ % по кредиту = ежемесячный взнос
            cell(monthly_fee);
            credit -= monthly_paid;
            cell(month < maturity ? credit : 0);
            table.append(row);
        }
    }
    function reset() {
        var list = location.hash ? location.hash.slice(1) : false;
        list = list ? list.split('|') : [];
        $('#amount').val(  list[0] || localStorage['amount']   || 5000);
        $('#interest').val(list[1] || localStorage['interest'] || 3);
        $('#maturity').val(list[2] || localStorage['maturity'] || 6);
        if (0 == list.length) {
            if ('year' == localStorage['interest_term'])
                $('#interest_term').val('year');
            if ('year' == localStorage['maturity_unit'])
                $('#maturity_unit').val('year');
        }
    }
    $(document).ready(function() {
        reset();
        $('#reset').click(reset);
        $('#credit').submit(function(e) {
            e.preventDefault();
            var interest = $('#interest').val();
            interest = interest.replace(',', '.');
            if ('year' == $('#interest_term').val())
                interest /= 12;
            var maturity = $('#maturity').val();
            if ('year' == $('#maturity_unit').val())
                maturity *= 12;
            calculate($('#amount').val(), interest, maturity);
            e.isDefaultPrevented = true;
        });
    });
    $(window).unload(function() {
        localStorage['amount'] = $('#amount').val();
        localStorage['interest'] = $('#interest').val();
        localStorage['maturity'] = $('#maturity').val();
        localStorage['interest_term'] = $('#interest_term').val();
        localStorage['maturity_unit'] = $('#maturity_unit').val();
    });
</script>
</body>
</html>
