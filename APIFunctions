//External Libraries
var request               = require('request'),
    mongoose              = require('mongoose'),
    HashMap               = require('hashmap'),
    groupBy               = require('group-by'),
    cachedRequest         = require('cached-request')(request),
// Internal Libraries 
    cacheDirectory        = "/tmp/cache",
    functionalAreaMap     = require('functionalAreaMap');


mongoose.connect("mongodb://localhost/brdemo");
cachedRequest.setCacheDirectory(cacheDirectory);

var userName = '';
var password = ''; //sanitized

//Class Scope Variables
var tmsTickets = [];
var ticketsForConcern = [];
var resBody;
var reportName;


////HashMap///

var ticketHashMap = new HashMap();
var orgHashMap = new HashMap();
var sideTable = new HashMap();

//***********
//DB Segment
//***********

var tableSchema = new mongoose.Schema({
    date: Date,
    url: String,
    data: Array,
    name: String
});

var brSchema = new mongoose.Schema({
    date:String,
    name:String,
    data: Array,
    protoData: Object
})

//Mongoose Schema Objects
var Table = mongoose.model("Table", tableSchema);
var BurnReport = mongoose.model("BurnReport", brSchema);


/////////////////////////////Functions//////////////////////////////////
var apiCalls ={

    projector:  function getProjectorData(url, callback){
            request(url,
                        function(err, res, body){
                            //local vars
                            var count = 0;
                            tmsTickets = [];
                            ticketsForConcern = [];
                            if (!err && res.statusCode == 200){
                                resBody = JSON.parse(body);
                                console.log('Projector Response ' + resBody.metadata.ReportName);
                                reportName = resBody.metadata.ReportName;
                                console.log('Project Response Size: ' + resBody.rows.length);
                                
                                resBody.rows.forEach(function(row){
                                    if(row.ProjectProjectType == 'TMS'){
                                        tmsTickets.push(row);
                                    }
                                    
                                    
                                    
                                }); 
                                 
                                 console.log("Number of Tickets " + tmsTickets.length);
                                //find entries with incorrect ticket numbers
                               //Complete this n times
                               tmsTickets.forEach(function(row){
                                
                                    //grab each ticket
                                    var query = row['TimeCardTMS-TicketNumber'];
                                    if(query == null || query == ''){
                                        count+=1;
                                        row.number = count;
                                        row.mismatch = 1;
                                        row.mismatchType = "Missing Ticket Number";
                                        ticketsForConcern.push(row);
                                    }
                                    
                                    else{
                                    //otherwise check ticket hashmap for org, if org name is not ticket org name  we have a problem//done locally 
                                    
                                    //querytickethashmap and org hashmap simultaneously
                                    console.log("Ticket: " + ticketHashMap.get(query) +" for query: " + query);
                                    
                                    //new case
                                    if(typeof(ticketHashMap.get(query)) === "undefined" || ticketHashMap.get(query) === null){
                                                    
                                                    count+=1;
                                                    row.number = count;
                                                    row.mismatch = 1;
                                                    row.mismatchType = "Ticket has not been updated since first day of year and is mismatched for reporting period";
                                                    row.src = "https://cpsgpartners.zendesk.com/agent/tickets/" + query;
                                                    ticketsForConcern.push(row);
                                                   
                                                }
                                if(typeof(ticketHashMap.get(query)) != "undefined"){                
                                    
                                    var orgName= orgHashMap.get(ticketHashMap.get(query).org);
                                    
                                    
                                    if(typeof(orgName) === "undefined"){
                                                    
                                                    count+=1;
                                                    row.number = count;
                                                    row.mismatch = 1;
                                                    row.mismatchType = "Ticket Number not in list of Open, Pending or On Hold Tickets";
                                                    row.src = "https://cpsgpartners.zendesk.com/agent/tickets/" + query;
                                                    ticketsForConcern.push(row);
                                                   
                                                }
                                    else if (!row.ClientName.includes(orgName) && !orgName.includes(row.ClientName && typeof(orgName) != "undefined")){
                                        
                                                    count+=1;
                                                    row.number = count;
                                                    row.mismatch = 1;
                                                    row.mismatchType = "Incorrect Ticket Number for Client";
                                                    row.src = "https://cpsgpartners.zendesk.com/agent/tickets/" + query;
                                                    ticketsForConcern.push(row);
                                                 }
                                                
                                    else{
                                                    row.mismatch = 0;
                                                
                                        }
                                        
                                } 
                                        
                                    }
                               }   
                                
                                   
                            );
                                     var currentDate = new Date();
                                     var newDataSet = {date: currentDate, url: url, data: ticketsForConcern, name: reportName}; //lets just make lots of datasets for now
                                     Table.create(newDataSet, function(err, table){
                                         if(err){
                                             throw Error(err);
                                         }
                                         
                                         else{
                                             console.log("Placed in DB: " + table.url + " at " + table.date);
                                            
                                         }
                                         
                                         callback();
                                     });
                                     
                            
                            }
                           
                        }
                    );
                    
                }   
        ,
        
    zendeskTickets: function getZendeskTicketData(url, callback){
                

                            request.get(url, function(err, res, body){
                                    if(err) 
                                    {
                                        throw Error('Something Went Wrong: ' + err);
                                    }
                                    
                                    else if(!err && res.statusCode == 200){
                                        //console.log('Request successful.');
                                        body = JSON.parse(body);
                                        //populate hashmap
                                        body.results.forEach(function(ticket){
                                             ticketHashMap.set(ticket.id, {org: ticket.organization_id, functionalArea: ticket.custom_fields[2].value});
                                        });
                                        //onsole.log("Next Page: " + body.next_page);
                                        }
                                        if(body.next_page != null){
                                            getZendeskTicketData(body.next_page, callback);
                                            //console.log(typeof callback)
                                        }
                                        
                                        else if ( body.next_page == null && typeof callback === 'function')
                                        {
                                            callback(ticketHashMap);
                                        }
                                    
                                
                                    }
                                    
                                 ).auth(userName, password, false);
                           
                            },
    
    zendeskOrgs: function getZendeskOrgData(url, callback){
                
                            //console.log("/n Top Level URL : " + url );
                            request.get(url, function(err, res, body){
                                    if(err) 
                                    {
                                        throw Error('Something Went Wrong: ' + err);
                                    }
                                    
                                    else if(!err && res.statusCode == 200){

                                        body = JSON.parse(body);
                                        //populate hashmap
                                        body.organizations.forEach(function(org){
                                            orgHashMap.set(org.id, org.name);
                                        });
                                        }
                                        if(body.next_page != null){
                                            getZendeskOrgData(body.next_page, callback);
                                            //console.log(typeof callback)
                                        }
                                        
                                        else if (body.next_page == null && typeof callback === 'function')
                                        {
                                            callback(orgHashMap);
                                        }
                                    
                                
                                    }
                                    
                                 ).auth(userName, password, false);
                           
                            },
                            
     projectBurnReport: function getProjectorBurnReports(url, callback){
         Date.prototype.monthNames = [
            "January", "February", "March",
            "April", "May", "June",
            "July", "August", "September",
            "October", "November", "December"
        ];
        
        Date.prototype.getMonthName = function() {
             return this.monthNames[this.getMonth()];
        };
        
        Date.prototype.getShortMonthName = function () {
            return this.getMonthName().substr(0, 3);
        };
        
        
        //monthly proto function
        Date.prototype.getMonthWeek = function(){
            var firstDay = new Date(this.getFullYear(), this.getMonth(), 1).getDay();
        return Math.ceil((this.getDate() + firstDay)/7) == 5? 1:Math.ceil((this.getDate() + firstDay)/7);};
        
        var options = {
            url: url,
            ttl:3000000000
        };
        
        //Maps For Functional Area

        cachedRequest(options,
                        function(err, res, body){
                            //local vars
                            tmsTickets = [];
                            ticketsForConcern = [];
                            if (!err && res.statusCode == 200){
                                resBody = JSON.parse(body);
                                console.log('Projector Response ' + resBody.metadata.ReportName);
                                reportName = resBody.metadata.ReportName;
                                var paramSummary = resBody.metadata.ParameterSummary;
                                console.log('Project Response Size: ' + resBody.rows.length);
                                
                                resBody.rows.forEach(function(row){
                                    if(row.ProjectName.includes('Production Services')){
                                        tmsTickets.push(row);
                                    }
                                    
                                    tmsTickets = tmsTickets;
                                    
                                });
                                

                                //get functional area from hashmap
                                
                                tmsTickets.forEach(function(ticket){
                                    ticket.weekNumber = new Date(ticket.Week).getMonthWeek();
                                    if(ticket["TimeCardTMS-TicketNumber"]!=null && ticket["TimeCardTMS-TicketNumber"]!="" && typeof (ticketHashMap.get(ticket["TimeCardTMS-TicketNumber"])) != "undefined" ){
                                        
                                        ticket.functionalArea = functionalAreaMap[ticketHashMap.get(ticket["TimeCardTMS-TicketNumber"]).functionalArea];
                                    }    
                                    else{
                                        ticket.functionalArea = 'General';
                                    }
                                    
                                });
                                
                                var tmsTicketsByClient = groupBy(tmsTickets, "ClientName");
                                
                                //group each client by functional area and each functional area by week and add the number of hours 
                                
                                for (var key in tmsTicketsByClient){
                                    
                                    //create hashmap for secondary table lookup
                                    console.log(key);
                                    console.log(tmsTicketsByClient[key][0]["ProjectTMS-HourAllotment"]);
                                    sideTable.set(key, {hoursAlloted: tmsTicketsByClient[key][0]["ProjectTMS-HourAllotment"], csm: tmsTicketsByClient[key][0].ProjectManagerDisplayName, dataMonth: new Date(tmsTicketsByClient[key][0].Week).getMonthName()});
                                    
                                    //group by functional area
                                    var value = groupBy(tmsTicketsByClient[key], "functionalArea");
                                    tmsTicketsByClient[key] = value;
                                    //group by week
                                    for (var ticketkey in tmsTicketsByClient[key]){
                                        var weekValue = groupBy(tmsTicketsByClient[key][ticketkey], "weekNumber");
                                        tmsTicketsByClient[key][ticketkey] = weekValue;
                                        
                                        
                                        //add hours together
                                        var sum = 0;
                                        for (var hoursKey in tmsTicketsByClient[key][ticketkey]){
                                            tmsTicketsByClient[key][ticketkey][hoursKey].forEach(function(instance){
                                                
                                                sum += instance.PersonHours;
                                                tmsTicketsByClient[key][ticketkey][hoursKey] = sum;
                                            }) }
                                        
                                    }
                            
                                     
                                }
                                
                                var i = 0; //to check last element
                                var length = Object.keys(tmsTicketsByClient).length;
                                
                                for (key in tmsTicketsByClient){
                                    //create client db item
                                    //need to check if Item Exists Prior to Placing 
                                    //get protodata for second table
                                     

                                     var newDataSet = {date: paramSummary, name: key, data: tmsTicketsByClient[key], protoData:sideTable.get(key)}; 
                                     console.log(tmsTicketsByClient[key]);//lets just make lots of datasets for now
                                     BurnReport.create(newDataSet, function(err, br){
                                         if(err){
                                             throw Error(err);
                                         }
                                         
                                         else{
                                             console.log("Placed in DB: " + br.name);
                                            
                                         }
                                         
                                         i++;
                                         
                                         if(i === length){
                                         //only callback if last element
                                         console.log("on last element");
                                         callback();
                                             
                                             
                                         }
                                         
                                     }
                                     
                                );
                            }
                    
                        }
                    }
                );
            }                        
        };

module.exports = {functions: apiCalls, model: {tableModel: Table, burnReportModel: BurnReport}};
