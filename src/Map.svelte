<script>

const CORS_PROXY_URL = "https://cors-anywhere.herokuapp.com";
const dataSource = 'https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/csse_covid_19_data/csse_covid_19_daily_reports';



let renderDate = null; 
let renderData = null; 

// format 00
function format(number) {
  if (number < 10) return '0' + number;
  return number;
}

// Convert to number
function number(data) {
  try {
    return parseInt(data, 10) || 0;
  } catch (error) {
    return 0;
  }
}

// Implement substract day function
Date.prototype.subtractDays = function(d) {  
  this.setTime(this.getTime() - (d*24*60*60*1000));  
  return this;  
} 

// Try fetching data from today, if failed try yesterday's
function fetchData(d) {
    console.log("fetch data")
  return new Promise((resolve, reject) => {
    const day = format(d.getDate());
    const month = format(d.getMonth() + 1);
    const year = d.getFullYear();
    const date = `${month}-${day}-${year}`;
    renderDate = `Date: ${date}`;
    renderData = 'Loading...';
    let fetchUrl = `${CORS_PROXY_URL}/${dataSource}/${date}.csv`;
    fetch(fetchUrl).then((response) => {
      return response && response.ok ? response.text() : fetchData(d.subtractDays(1));;
    }).then((text) => {
      return resolve(text);
    }).catch((error) => {
      console.error(error)
      console.error(fetchUrl)

      reject(error);
    });
  });
}


async function main() {
  try {
    let d = new Date();
    let data = await fetchData(d);
    // Column Index:0              ,1             ,2                  ,3        ,4     ,5        ,6       ,7
    // Province/State              ,Country/Region,Last Update        ,Confirmed,Deaths,Recovered,Latitude,Longitude
    // Hubei                       ,Mainland China,2020-03-02T15:03:23,67103    ,2803  ,33934    ,30.9756 ,112.2707
    //                             ,South Korea   ,2020-03-02T20:23:16,4335     ,28    ,30       ,36.0000 ,128.0000
    // "Santa Clara, CA"           ,US            ,2020-03-02T20:33:02,9        ,0     ,1        ,37.3541 ,-121.9552
    // Diamond Princess cruise ship,Others        ,2020-03-03T03:13:06,706      ,6     ,10       ,35.4437 ,139.6380

    console.log(data);
    data = data.split("\n");
    const PROVINCE = 0;
    const COUNTRY = 1;
    const CONFIRMED = 3;
  
    let countries = new Set()
    let countryConfirmedCases = [];
    for(let i=1; i<data.length; i++) {
      let items = data[i].split(',');
      let extraIndex = items.length - 8;
      let country = items[COUNTRY+extraIndex];
      const confirmedCase = items[CONFIRMED+extraIndex];
      if (country === 'Others') country = `Others - ${items[PROVINCE]}`;
      if (!country || !confirmedCase) continue;
      countries.add(country);
      countryConfirmedCases[country] = number(countryConfirmedCases[country]) + number(confirmedCase);
    }
    
    let result = '';
    countries.forEach(country => {
      let confirmedCase = number(countryConfirmedCases[country]);
      result += `
        <li>
          ${country}: ${confirmedCase}
        </li>
      `;
    });
    result = `
      <ul>
        ${result}
      </ul>
    `;
    renderData = result;
  } catch (error) {
      console.error(error)
    renderData = 'Could not load data. Please try again!';
  }
}

main();

</script>

<h1 id="date">Coronavirus statistics by the Johns Hopkins University Center for Systems Science and Engineering</h1>
<div id="data">{@html renderDate}</div>
<div id="data">{@html renderData}</div>
<p>This is the data repository for the 2019 Novel Coronavirus Visual Dashboard operated by the Johns Hopkins University Center for Systems Science and Engineering (JHU CSSE). Also, Supported by ESRI Living Atlas Team and the Johns Hopkins University Applied Physics Lab (JHU APL).</p>

