import sys
import os
import getopt
import xlrd
import geocoder
import datetime
from MySQLConnector import Connection

def log(program, entry, error):
    program = program
    entry = entry
    error = error
    command = "python3 logger.py -p %s -e %s -r %s" % (program, entry, error)
    return command

os.system(log("\"SQL_RBQSITES.PY\"", "\"Opening SQL connection...\"", ""))
try:
    from MySQLConnector import Connection
    cursor, conn = Connection()
    os.system(log("\"SQL_RBQSITES.PY\"", "\"Connection Successful\"", ""))
except Exception as err:
    print(err)
    os.system(log("\"SQL_RBQSITES.PY\"", "", "\"Failed opening SQL connection\""))


def main(argv):
    inputfile = ''
    outputfile = ''
    try:
        opts, args = getopt.getopt(argv, "hi:o:", ["ifile=", "ofile="])
    except getopt.GetoptError:
        print('test.py -i <inputfile> -o <outputfile>')
        sys.exit(2)
    for opt, arg in opts:
        if opt == '-h':
            print('test.py -i <inputfile> -o <outputfile>')
            sys.exit()
        elif opt in ("-i", "--ifile"):
            inputfile = arg
        elif opt in ("-o", "--ofile"):
            outputfile = arg
    print('Input file is ', inputfile)
    os.system(log("\"SQL_RBQSITES.PY:MAIN\"", "\"Outputfile is\"" + inputfile + "", ""))
    print('Output file is ', outputfile)
    os.system(log("\"SQL_RBQSITES.PY:MAIN\"", "\"Outputfile is\"" + outputfile + "", ""))
    book = xlrd.open_workbook(inputfile)
    # Print book and number of sheets
    print(book.nsheets)
    os.system(log("\"SQL_RBQSITES.PY:MAIN\"", "\"Number of sheets is: \"" + book.nsheets + "", ""))
    print(book.sheet_names())
    os.system(log("\"SQL_RBQSITES.PY:MAIN\"", "\"Book name: \"" + book.sheet_names() + "", ""))
    first_sheet = book.sheet_by_index(0)
    try:
        os.system(log("\"SQL_RBQSITES.PY:MAIN\"", "\"Iterating...\"", ""))
        # iterate row and extract 6 columns: reg mun imm typ rue dos
        for row in range(first_sheet.nrows):
            if row > 1:
                reg = first_sheet.cell(row, 0).value
                regtyp = first_sheet.cell(row, 0).ctype
                mun = first_sheet.cell(row, 1).value
                muntyp = first_sheet.cell(row, 1).ctype
                # cast as int to remove . or take as string for example when contains 435-A
                try:
                    imm = int(first_sheet.cell(row, 2).value)
                    immtyp = first_sheet.cell(row, 2).ctype
                except ValueError:
                    imm = first_sheet.cell(row, 2).value
                immstr = str(imm)
                # immstr1, immstr2 = immstr.split('.')
                typ = first_sheet.cell(row, 3).value
                typtyp = first_sheet.cell(row, 3).ctype
                ruetyp = first_sheet.cell(row, 4).ctype
                if ruetyp == 2:  # int
                    rue = int(first_sheet.cell(row, 4).value)
                    ruestr = str(rue)
                elif ruetyp == 1:
                    rue = first_sheet.cell(row, 4).value
                    ruestr = rue
                # cast as int to remove .
                try:
                    dos = int(first_sheet.cell(row, 5).value)
                    dostyp = first_sheet.cell(row, 5).ctype
                except ValueError:
                    dos = first_sheet.cell(row, 5).value
                dosstr = str(dos)
                # adress = str(imm + " " + typ + " " + rue + ", " + mun + ", QC"
                print(reg, regtyp, mun, muntyp, immstr, immtyp, typ, typtyp, rue, ruetyp, dosstr, dostyp)
                # cursor = conn.cursor(buffered=True)
                # IF NOT DOS NOT FOUND IN RBQSITES INSERT ELSE UPDATE
                sqlSelectDos = "SELECT * FROM contaminated_geoindex_xyz.RBQ WHERE dos LIKE \"%s\"" % (dos)
                print(sqlSelectDos)
                cursor.execute(sqlSelectDos)
                if cursor.rowcount > 0:
                    print("FOUND: %s TODO UPDATE" % (dos))
                else:
                    print("NOT FOUND INSERTING IF GEOREFERENCED")
                    adress = immstr + " " + typ + " " + ruestr + ", " + mun + ", QC"
                    adress = adress.replace("\"", "'")
                    # IF ADRESS NOT FOUND IN ADRESSES GEOCODE ADRESS AND INSERT INTO ADRESSES
                    print(adress)
                    sqlSelectAdress = "SELECT contaminated_geoindex_xyz.ADRESSES.ADRESSE, contaminated_geoindex_xyz.ADRESSES.LONGITUDE, contaminated_geoindex_xyz.ADRESSES.LATITUDE FROM contaminated_geoindex_xyz.ADRESSES WHERE LENGTH(contaminated_geoindex_xyz.ADRESSES.LONGITUDE) > 0 AND contaminated_geoindex_xyz.ADRESSES.ADRESSE LIKE \"%s\"" % (
                    adress)
                    print(sqlSelectAdress)
                    cursor.execute(sqlSelectAdress)
                    if cursor.rowcount > 0:
                        data = cursor.fetchone()
                        longitude = data[1]
                        latitude = data[2]
                        print("FOUND: %s %s" % (longitude, latitude))
                    else:
                        print("NOT FOUND GEOCODING")
                        g = geocoder.google(adress)
                        if g.latlng is not None:
                            lat = g.latlng[0]
                            lon = g.latlng[1]
                            latitude = str(lat)
                            longitude = str(lon)
                            print("%s %s" % (longitude, latitude))
                        else:
                            latitude = ""
                            longitude = ""
                            print("NO ADRESS FOUND...")
                        sqlInsertAdress = "INSERT INTO contaminated_geoindex_xyz.ADRESSES (ADRESSE,LONGITUDE,LATITUDE) VALUES (\"%s\",\"%s\",\"%s\")" % (
                        adress, longitude, latitude)
                        print(sqlInsertAdress)
                        cursor.execute(sqlInsertAdress)
                        conn.commit()
                    if len(latitude) > 0:
                        sqlInsertRbq = "INSERT INTO contaminated_geoindex_xyz.RBQ (reg,mun,immeuble,typrue,rue,dos,longitude,latitude) VALUES (\"%s\",\"%s\",\"%s\",\"%s\",\"%s\",\"%s\",\"%s\",\"%s\")" % (
                        reg, mun, immstr, typ, rue, dosstr, longitude, latitude)
                        print(sqlInsertRbq)
                        cursor.execute(sqlInsertRbq)
                        conn.commit()
                    else:
                        print("DISCARDING, TODO LOG")
        sqlDeleteSitesRBQ = "DELETE FROM SITES WHERE SOURCE LIKE \"RBQ\""
        cursor.execute(sqlDeleteSitesRBQ)
        conn.commit()
        sqlInsertSitesRBQ = "INSERT INTO SITES (AFFICH,COORD,SOURCE) SELECT CONCAT(dos,'\n',immeuble,' ',typrue, ' ', rue, ' ',mun,' ', reg,'\n',cap_aur,' ',nb_autoris), ST_GeomFromText(CONCAT('POINT(',REPLACE(LATITUDE,',','.'),' ',REPLACE(LONGITUDE,',','.'),')')), 'RBQ' FROM RBQ"
        cursor.execute(sqlInsertSitesRBQ)
        sqlRecordCount = "SELECT COUNT(*) FROM RBQ"
        cursor.execute(sqlRecordCount)
        record_count = cursor.fetchone()
        record_countstr = str(record_count)
        timestamp = datetime.datetime.utcnow()
        sqlUpdate = "UPDATE contaminated_geoindex_xyz.SOURCES SET LAST_UPDATE=\"%s\",RECORD_COUNT=%s" % (
        timestamp, record_countstr)
        cursor.execute(sqlUpdate)
        conn.commit()
        cursor.close()
    except Exception as err:
        print(err)
    else:
        conn.close()


if __name__ == "__main__":
    main(sys.argv[1:])
