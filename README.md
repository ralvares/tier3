### Namespace per tier ###

oc new-project kiosk-backend
oc new-project kiosk-frontend
oc new-project kiosk-database


oc new-app https://github.com/jankleinert/concession-kiosk-backend --name backend -n kiosk-backend


oc new-app https://github.com/jankleinert/concession-kiosk-frontend --name frontend -n kiosk-frontend -e COMPONENT_BACKEND_HOST=backend.kiosk-backend -e COMPONENT_BACKEND_PORT=8080

oc expose svc/frontend -n kiosk-frontend

oc get route -n kiosk-frontend


oc apply -f mongodb-ephemeral.json -n kiosk-database
oc new-app --template=mongodb-ephemeral -n kiosk-database --param=MONGODB_USER=mongodb --param=MONGODB_PASSWORD=mongodb --param=DATABASE_SERVICE_NAME=database
oc set env deployment/backend MONGODB_URL=mongodb://mongodb:mongodb@database.kiosk-database/sampledb -n kiosk-backend


From database to frontend

curl http://frontend.kiosk-frontend:8080
curl http://backend.kiosk-backend:8080


Apply network rules and test again.

Create Service Mesh
Add members - show network rules and try to access.. 

### RBAC ###

User - frontend-dev 
User - backend-dev 
User - mongodbadmin 

Group - developers - front and backend 
Group dbadmins - mongodbadmin

Group developers view access to frontend and backend

frontend-dev user admin to frontend
backend-dev user admin to backend
group dbadmins -> admin to database


# Create restricte Roles #
oc create clusterrole podview --verb=get,list --resource=pod,pods/log
oc create clusterrole nsview --verb=get,list --resource=namespaces,projects
oc create clusterrole eventsview --verb=get,list,watch --resource=events

# Adding users to groups #
oc adm groups new developers
oc adm groups add-users developers frontend-dev backend-dev
oc adm groups new dbadmins
oc adm groups add-users dbadmins mongodbadmin

# Binding Roles to groups #
oc adm policy add-role-to-group podview developers -n kiosk-frontend
oc adm policy add-role-to-group podview developers -n kiosk-backend
oc adm policy add-role-to-group admin dbadmins -n kiosk-database

# Binding role to user #

oc adm policy add-role-to-user edit frontend-dev -n kiosk-frontend
oc adm policy add-role-to-user edit backend-dev -n kiosk-backend

# Tests #

# Testing user frontend-dev

# listing pods #
oc auth can-i --as=frontend-dev --as-group=developers get pod -n kiosk-frontend
oc auth can-i --as=frontend-dev --as-group=developers get pod -n kiosk-backend
oc auth can-i --as=frontend-dev --as-group=developers get pod -n kiosk-database

# delete pods #
oc auth can-i --as=frontend-dev --as-group=developers delete pod -n kiosk-frontend
oc auth can-i --as=frontend-dev --as-group=developers delete pod -n kiosk-backend
oc auth can-i --as=frontend-dev --as-group=developers delete pod -n kiosk-database

# Testing user backend-dev

# listing pods #
oc auth can-i --as=backend-dev --as-group=developers get pod -n kiosk-frontend
oc auth can-i --as=backend-dev --as-group=developers get pod -n kiosk-backend
oc auth can-i --as=backend-dev --as-group=developers get pod -n kiosk-database

# delete pods #
oc auth can-i --as=backend-dev --as-group=developers delete pod -n kiosk-frontend
oc auth can-i --as=backend-dev --as-group=developers delete pod -n kiosk-backend
oc auth can-i --as=backend-dev --as-group=developers delete pod -n kiosk-database


# Testing user dbadmins

# listing pods #
oc auth can-i --as=mongodbadmin --as-group=dbadmins get pod -n kiosk-frontend
oc auth can-i --as=mongodbadmin --as-group=dbadmins get pod -n kiosk-backend
oc auth can-i --as=mongodbadmin --as-group=dbadmins get pod -n kiosk-database

# delete pods #
oc auth can-i --as=mongodbadmin --as-group=dbadmins delete pod -n kiosk-frontend
oc auth can-i --as=mongodbadmin --as-group=dbadmins delete pod -n kiosk-backend
oc auth can-i --as=mongodbadmin --as-group=dbadmins delete pod -n kiosk-database


## Delete all ##

oc delete project kiosk-backend 
oc delete project kiosk-frontend
oc delete project kiosk-database
