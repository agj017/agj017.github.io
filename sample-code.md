

2023-04-07 19:26:58.010  INFO 93575 --- [    Test worker] .b.d.b.u.CreateBatteryTransactionUseCase : battery movements for when KOABDB0AX2970YD8D: [BatteryMovementDto(batteryId=test-battery-1846205, from=HolderId(id=RDXGK26VEW1G, type=VEHICLE, name=VEHICLE(EVID: VEW1G VIN: KOABDB0AX2970YD8D), location=GeoPoint(lat=0.0, lng=0.0)), to=HolderId(id=test-station-cf83c7ef-c890-4627-8a17-b28595ea2d19, type=STATION, name=test-station-cf83c7ef-c890-4627-8a17-b28595ea2d19, location=GeoPoint(lat=0.0, lng=0.0))), BatteryMovementDto(batteryId=test-battery-9754127, from=HolderId(id=RDXGK26VEW1G, type=VEHICLE, name=VEHICLE(EVID: VEW1G VIN: KOABDB0AX2970YD8D), location=GeoPoint(lat=0.0, lng=0.0)), to=HolderId(id=test-station-cf83c7ef-c890-4627-8a17-b28595ea2d19, type=STATION, name=test-station-cf83c7ef-c890-4627-8a17-b28595ea2d19, location=GeoPoint(lat=0.0, lng=0.0))), BatteryMovementDto(batteryId=test-battery-8237617, from=HolderId(id=RDXGK26VEW1G, type=VEHICLE, name=VEHICLE(EVID: VEW1G VIN: KOABDB0AX2970YD8D), location=GeoPoint(lat=0.0, lng=0.0)), to=HolderId(id=test-station-cf83c7ef-c890-4627-8a17-b28595ea2d19, type=STATION, name=test-station-cf83c7ef-c890-4627-8a17-b28595ea2d19, location=GeoPoint(lat=0.0, lng=0.0))), BatteryMovementDto(batteryId=test-battery-4452777, from=HolderId(id=test-station-cf83c7ef-c890-4627-8a17-b28595ea2d19, type=STATION, name=test-station-cf83c7ef-c890-4627-8a17-b28595ea2d19, location=GeoPoint(lat=0.0, lng=0.0)), to=HolderId(id=RDXGK26VEW1G, type=VEHICLE, name=VEHICLE(EVID: VEW1G VIN: KOABDB0AX2970YD8D), location=GeoPoint(lat=0.0, lng=0.0))), BatteryMovementDto(batteryId=test-battery-1703756, from=HolderId(id=test-station-cf83c7ef-c890-4627-8a17-b28595ea2d19, type=STATION, name=test-station-cf83c7ef-c890-4627-8a17-b28595ea2d19, location=GeoPoint(lat=0.0, lng=0.0)), to=HolderId(id=RDXGK26VEW1G, type=VEHICLE, name=VEHICLE(EVID: VEW1G VIN: KOABDB0AX2970YD8D), location=GeoPoint(lat=0.0, lng=0.0))), BatteryMovementDto(batteryId=test-battery-9035778, from=HolderId(id=test-station-cf83c7ef-c890-4627-8a17-b28595ea2d19, type=STATION, name=test-station-cf83c7ef-c890-4627-8a17-b28595ea2d19, location=GeoPoint(lat=0.0, lng=0.0)), to=HolderId(id=RDXGK26VEW1G, type=VEHICLE, name=VEHICLE(EVID: VEW1G VIN: KOABDB0AX2970YD8D), location=GeoPoint(lat=0.0, lng=0.0)))]
2023-04-07 19:27:00.053  INFO 93575 --- [   scheduling-1] i.m.o.b.i.BatteryServiceScheduler        : handle offline batteries start
2023-04-07 19:27:10.085  WARN 93575 --- [   scheduling-1] o.h.engine.jdbc.spi.SqlExceptionHelper   : SQL Error: 0, SQLState: 57014
2023-04-07 19:27:10.085 ERROR 93575 --- [   scheduling-1] o.h.engine.jdbc.spi.SqlExceptionHelper   : ERROR: canceling statement due to user request
2023-04-07 19:27:10.105 ERROR 93575 --- [   scheduling-1] o.s.s.s.TaskUtils$LoggingErrorHandler    : Unexpected error occurred in scheduled task

org.springframework.dao.QueryTimeoutException: could not extract ResultSet; SQL [n/a]; nested exception is org.hibernate.QueryTimeoutException: could not extract ResultSet

미국 이민관련: https://velog.io/@diogenes0803/%EB%AF%B8%EA%B5%AD-%EA%B0%9C%EB%B0%9C%EC%9E%90%EA%B0%80-%EC%95%8C%EB%A0%A4%EC%A3%BC%EB%8A%94-%EB%82%98%EB%8F%84-%ED%95%A0%EC%88%98%EC%9E%88%EB%8B%A4-%EB%AF%B8%EA%B5%AD%EC%9C%A0%ED%95%99 -> https://www.changbal.org/


@Test fun에서 @Transactional이 있는 경우 @DataLoader를 호출하면 시스템이 멈춤 이유는 뭘까? 아래와 같은 코드
findVehicleUseCase.findAllByVinIn(nonNullKeys.toList()) -> DataLoader에서 디비 접근할경우 멈춤

    @Test
    fun `test`() {
        testHelpers.deleteAllFrom("vehicle")
        testHelpers.deleteAllFrom("battery")
        val now = Instant.now()

        val swappedVehicle = testHelpers.createVehicle(
            latestPacketReceivedAt = now,
            withBatteryLevel = 100f
        )
        // prepare a paid transaction
        val batteryTransaction = testHelpers.createCustomerSwapBatteryTransaction(vehicle = swappedVehicle)
        setBatteryTransactionPaidUseCase.execute(
            batteryTransactions = listOf(batteryTransaction),
            paymentTransactionId = UUID.randomUUID(),
            payerId = UUID.randomUUID(),
            paidAmount = BigDecimal.ZERO,
        )

        val vehicle = gqlTestHelpers.getVehicle(
            GetVehicleInput(
                id = swappedVehicle.id
            )
        )

        println("hello")
    }